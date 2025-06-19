# rtty - 在任何地方通过Web访问您的设备的终端

[1]: https://img.shields.io/badge/开源协议-MIT-brightgreen.svg?style=plastic
[2]: /LICENSE
[3]: https://img.shields.io/badge/提交代码-欢迎-brightgreen.svg?style=plastic
[4]: https://github.com/zhaojh329/rtty/pulls
[5]: https://img.shields.io/badge/提问-欢迎-brightgreen.svg?style=plastic
[6]: https://github.com/zhaojh329/rtty/issues/new
[10]: https://raw.githubusercontent.com/CodePhiliaX/resource-trusteeship/main/readmex.svg
[11]: https://readmex.com/zhaojh329/rtty
[12]: https://deepwiki.com/badge.svg
[13]: https://deepwiki.com/zhaojh329/rtty
[14]: https://img.shields.io/badge/技术交流群-点击加入：153530783-brightgreen.svg
[15]: https://jq.qq.com/?_wv=1027&k=5PKxbTV

[![license][1]][2]
[![PRs Welcome][3]][4]
[![Issue Welcome][5]][6]
[![ReadmeX][10]][11]
[![Ask DeepWiki][12]][13]
![visitors](https://visitor-badge.laobi.icu/badge?page_id=zhaojh329.rtty)
[![Chinese Chat][14]][15]

[Xterm.js]: https://github.com/xtermjs/xterm.js
[vue]: https://github.com/vuejs/vue

```mermaid
flowchart TB
s[rttys with public IP address]
u1["User(Web Browser)"] --> s
u2["User(Web Browser)"] --> s
u3["User(Web Browser)"] --> s
s --> c1["rtty(Linux Device)"]
s --> c2["rtty(Linux Device)"]
s --> c3["rtty(Linux Device)"]
```

![](/img/terminal.gif)
![](/img/file.gif)
![](/img/web.gif)

它由客户端和服务端组成。客户端采用纯C实现。服务端采用GO语言实现，前端界面采用[vue]实现。

您可以在任何地方通过Web访问您的设备的终端。通过设备ID来区分您的不同的设备。

rtty非常适合远程维护您的或者您的公司的部署在全球各地的成千上万的Linux设备。

## 特性
* 客户端 C 语言实现，非常小，适合嵌入式 Linux
  - 不支持 SSL: rtty(32K) + libev(56K)
  - 支持 SSL: + libmbedtls(88K) + libmbedcrypto(241K) + libmbedx509(48k)
* 远程批量执行命令
* SSL 双向认证(mTLS)
* 非常方便的上传和下载文件
* 根据设备ID访问不同的设备
* 支持 HTTP/HTTPS/WebSocket 代理 - 访问您的设备的 Web
* 基于[Xterm.js]的全功能终端
* 部署简单，使用方便

## 谁在使用 rtty
- [深圳市广联智通科技有限公司](https://www.gl-inet.com/)
- [深圳市云联芯科技有限公司](http://www.iyunlink.com/)
- [成都四海万联科技有限公司](https://www.oneiotworld.com/)
- [bitswrt Communication Technology](http://bitswrt.com/)

## Star History
[![Star History Chart](https://api.star-history.com/svg?repos=zhaojh329/rtty&type=Date)](https://www.star-history.com/#zhaojh329/rtty&Date)

## Star History
[![Star History Chart](https://api.star-history.com/svg?repos=zhaojh329/rtty&type=Date)](https://www.star-history.com/#zhaojh329/rtty&Date)

## 贡献代码
如果您想帮助 rtty 变得更好，请参考 [CONTRIBUTING_ZH.md](/CONTRIBUTING_ZH.md)。

## 强烈推荐佐大的 OpenWrt 培训班
想学习 OpenWrt 开发，但是摸不着门道？自学没毅力？基础太差？怕太难学不会？快来参加<跟着佐大学 OpenWrt 开发入门培训班> 佐大助你能学有所成，培训班报名地址：http://forgotfun.org/2018/04/openwrt-training-2018.html
