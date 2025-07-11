<template>
  <div>
    <div ref="terminal" :style="{height: termHeight + 'px', margin: '5px'}" @contextmenu.prevent="showContextmenu"/>
    <el-dialog v-model="file.modal" :title="$t('Upload file to device')" @close="onUploadDialogClosed" :width="400">
      <el-upload :before-upload="beforeUpload" action="#">
        <el-button type="primary">{{ $t("Select file") }}</el-button>
      </el-upload>
      <p v-if="file.file !== null"> {{ file.file.name }}</p>
      <template #footer>
        <el-button @click="file.modal = false">{{ $t('Cancel') }}</el-button>
        <el-button type="primary" @click="doUploadFile">{{ $t('OK') }}</el-button>
      </template>
    </el-dialog>
    <contextmenu ref="contextmenu" :menus="contextmenus" @click="onContextmenuClick"/>
    <el-dialog v-model="font.modal" :show-close="false" :width="180" header-class="font-size-dialog-header">
      <el-input-number v-model="font.size" :min="10" :max="30" @change="updateFontSize"/>
    </el-dialog>
  </div>
</template>

<script>
import { Terminal } from '@xterm/xterm'
import { FitAddon } from '@xterm/addon-fit'
import '@xterm/xterm/css/xterm.css'

import OverlayAddon from '../xterm-addon/xterm-addon-overlay'
import Contextmenu from '../components/ContextMenu.vue'

const LoginErrorOffline = 0x01
const LoginErrorBusy = 0x02

const MsgTypeFileData = 0x03

const ReadFileBlkSize = 63 * 1024

const AckBlkSize = 4 * 1024

export default {
  name: 'Rtty',
  components: {
    'Contextmenu': Contextmenu
  },
  props: {
    devid: String
  },
  data() {
    return {
      contextmenus: [
        {name: 'copy', caption: this.$t('Copy - Ctrl+Insert')},
        {name: 'paste', caption: this.$t('Paste - Shift+Insert')},
        {name: 'clear', caption: this.$t('Clear Scrollback')},
        {name: 'font', caption: this.$t('Font Size')},
        {name: 'file', caption: this.$t('Upload or download file')},
        {name: 'about', caption: this.$t('About')}
      ],
      font: {
        modal: false,
        size: 16
      },
      file: {
        modal: false,
        accepted: false,
        file: null,
        offset: 0,
        fr: new FileReader()
      },
      disposables: [],
      resizeDelay: null,
      termHeight: 0,
      socket: null,
      term: null,
      fitAddon: null,
      sid: '',
      unack: 0
    }
  },
  methods: {
    showContextmenu(e) {
      this.$refs.contextmenu.show(e)
    },
    onContextmenuClick(name) {
      if (name === 'copy') {
        const text = this.term.getSelection()
        if (text) {
          this.$copyText(text).then(() => {
            this.$message.info(this.$t('Already copied to clipboard'))
          })
        }
      } else if (name === 'paste') {
        this.$message.info(this.$t('Please use shortcut "Shift+Insert"'))
      } else if (name === 'clear') {
        this.term.clear()
      } else if (name === 'font') {
        this.font.modal = true
      } else if (name === 'file') {
        this.$message.info(this.$t('Please execute command "rtty -R" or "rtty -S" in current terminal!'))
      } else if (name === 'about') {
        window.open('https://github.com/zhaojh329/rtty')
      }

      this.term.focus()
    },
    updateFontSize(size) {
      if (!size) {
        size = 16
        this.font.size = 16
      }

      this.term.options.fontSize = size
      this.fitAddon.fit()
    },
    onUploadDialogClosed() {
      this.term.focus()
      if (this.file.accepted)
        return
      this.file.file = null
      const msg = {type: 'fileCanceled'}
      this.socket.send(JSON.stringify(msg))
    },
    beforeUpload(file) {
      this.file.file = file
      return false
    },
    submitUploadFile() {
      this.$refs.upload.submit()
    },
    sendFileInfo(file) {
      const msg = {type: 'fileInfo', size: file.size, name: file.name}
      this.socket.send(JSON.stringify(msg))
    },
    readFileBlob(fr, file, offset, size) {
      const blob = file.slice(offset, offset + size)
      fr.readAsArrayBuffer(blob)
    },
    doUploadFile() {
      if (!this.file.file) {
        this.onUploadDialogClosed()
        return
      }

      this.term.focus()

      if (this.file.size > 0xffffffff) {
        this.$message.error(this.$t('The file you will upload is too large(> 4294967295 Byte)'))
        return
      }

      this.file.accepted = true
      this.file.modal = false

      this.sendFileInfo(this.file.file)

      if (this.file.size === 0) {
        this.sendFileData(null)
        return
      }

      this.file.offset = 0

      const fr = this.file.fr

      fr.onload = e => {
        this.file.offset += e.loaded
        this.sendFileData(new Uint8Array(fr.result))
      }
      this.readFileBlob(fr, this.file.file, this.file.offset, ReadFileBlkSize)
    },
    sendTermData(data) {
      this.socket.send(new Uint8Array([0, ...new TextEncoder().encode(data)]))
    },
    sendFileData(data) {
      let b

      if (data !== null)
        b = new Uint8Array([1, MsgTypeFileData, ...data])
      else
        b = new Uint8Array([1, MsgTypeFileData])

      this.socket.send(b)
    },
    fitTerm() {
      this.termHeight = document.documentElement.clientHeight - 10

      this.$nextTick(() => {
        if (this.resizeDelay)
          clearTimeout(this.resizeDelay)
        this.resizeDelay = setTimeout(() => {
          this.fitAddon.fit()
        }, 200)
      })
    },
    closed() {
      this.term.write('\n\n\r\x1B[1;3;31mConnection is closed.\x1B[0m')
      this.dispose()
    },
    openTerm() {
      const term = new Terminal({
        cursorBlink: true,
        fontSize: this.font.size
      })
      this.term = term

      const fitAddon = new FitAddon()
      this.fitAddon = fitAddon
      term.loadAddon(fitAddon)

      const overlayAddon = new OverlayAddon()
      term.loadAddon(overlayAddon)

      term.open(this.$refs['terminal'])
      term.focus()

      this.disposables.push(term.onData(data => this.sendTermData(data)))
      this.disposables.push(term.onBinary(data => this.sendTermData(data)))

      this.disposables.push(term.onResize(size => {
        const msg = {type: 'winsize', cols: size.cols, rows: size.rows}
        this.socket.send(JSON.stringify(msg))
        overlayAddon.show(term.cols + 'x' + term.rows)
      }))

      window.addEventListener('resize', this.fitTerm)

      this.disposables.push({
        dispose: () => window.removeEventListener('resize', this.fitTerm)
      })

      this.fitTerm()
    },
    dispose() {
      this.disposables.forEach(d => d.dispose())
    }
  },
  mounted() {
    const protocol = (location.protocol === 'https:') ? 'wss://' : 'ws://'

    const socket = new WebSocket(protocol + location.host + `/connect/${this.devid}`)
    socket.binaryType = 'arraybuffer'
    this.socket = socket

    socket.addEventListener('message', ev => {
      const data = ev.data

      if (typeof data === 'string') {
        const msg = JSON.parse(data)
        if (msg.type === 'login') {
          if (msg.err === LoginErrorOffline) {
            this.$router.push('/error/offline')
            return
          } else if (msg.err === LoginErrorBusy) {
            this.$router.push('/error/full')
            return
          }

          this.sid = msg.sid

          this.openTerm()

          socket.addEventListener('close', () => this.closed())
          socket.addEventListener('error', () => this.closed())
        } else if (msg.type === 'sendfile') {
          const el = document.createElement('a')
          el.style.display = 'none'
          el.href = '/file/' + this.sid
          el.download = msg.name
          el.click()
        } else if (msg.type === 'recvfile') {
          this.file.modal = true
          this.file.file = null
          this.file.accepted = false
          this.term.blur()
        } else if (msg.type === 'fileAck') {
          if (this.file.file && this.file.offset < this.file.file.size)
            this.readFileBlob(this.file.fr, this.file.file, this.file.offset, ReadFileBlkSize)
        }
      } else {
        const data = new Uint8Array(ev.data)
        this.unack += data.length
        this.term.write(data)

        if (this.unack > AckBlkSize) {
          const msg = {type: 'ack', ack: this.unack}
          socket.send(JSON.stringify(msg))
          this.unack = 0
        }
      }
    })
  },
  unmounted() {
    this.dispose()
    this.term.dispose()
  }
}
</script>

<style scoped>
  .xterm .xterm-viewport {
    overflow: auto;
  }

  :deep(.font-size-dialog-header) {
    display: none;
  }
</style>
