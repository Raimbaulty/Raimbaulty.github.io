# Cloudflare 免费 CDN

开始之前，需要准备2个域名并托管到 Cloudflare

- 域名一，回退域名，可以使用价格低廉的6位数字xyz域名，下面以 `123456.xyz`为例
- 域名二，使用域名，建议使用正规的com、net、org域名，下面以 `zhenggui.com` 为例

1. 回退域名DNS

打开`https://dash.cloudflare.com/?to=/:account/123456.xyz/dns/records`，注意替换回退域名

添加A记录，名称 `origin`，内容填写服务器IP地址，并确保打开小黄云

添加CNAME记录，名称 `cdn`，内容填写优选域名，[点击这里](https://www.wetest.vip/page/cloudflare/cname.html) 获取优选域名，注意关闭小黄云

2. 回退域名自定义域

打开：`https://dash.cloudflare.com/?to=/:account/123456.xyz/ssl-tls/custom-hostnames`，注意替换回退域名

如果没有支付方式会要求绑定支付方式，选择Free计划即可，然后点击 **添加回退源**，填写 `origin.1234567.xyz`

接着点击 **添加自定义主机名**，填写 `zhenggui.com`，点击 ▼，查看提示

3. 正规域名DNS

替换正规域名后在新标签页打开：`https://dash.cloudflare.com/?to=/:account/zhenggui.com/dns/records`

按照上面的提示添加 TXT记录

然后再添加一条CNAME记录，名称 `@`，指向 `origin.123456.xyz`，注意关闭小黄云

接着回到第二步打开的页面，点击刷新等待验证生效后，将上一步添加的CNAME记录值由 `origin.123456.xyz` 改为 `cdn.123456.xyz`

[点击这里](https://www.itdog.cn/) 输入域名后，点击单次查询查看连接情况