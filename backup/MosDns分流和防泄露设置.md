mosdns是一个不错的Dns转发插件，可以很方便的配置转发DNS查询的规则。
在我们正常使用Clash的时候，由于存在IP匹配规则，当你访问一个域名的时候不知道这个域名对应的IP是否满足这条分流规则，于是就会对上游DNS服务器进行一次请求，这里就导致了运营商知道你在访问哪些网站。
而mosdns有自己的分流规则，会分离国内/国外的查询请求走不同的服务器，同时不在他库里的可以强制走国外服务器查询DNS，以此来避免DNS泄露。
同时，mosdns拥有缓存，命中了缓存服务器后的DNS回应是本地回应，速度很快。
**下面是配置部分**
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/c802c036-e5e5-4cf7-bc52-faad9b0758a2)
在shellcrash中，我们关闭DNS拦截请求，我们不需要这个软件去拦截DNS请求了，也不需要什么fake-ip了，一切请求交给mosdns来完成。
同时，在shellcrash中，需要将代理模式改到TUN模式，因为mosdns向上游查询的规则是国外的服务器，可能受到运营商的拦截，这里直接选用IP的话我们可以让这个查询规则命中shellcrash的分流规则，走我们的VPS出去解析返回正确的结果。同理，国外域名解析的话我们最好直接使用IP的形式，为了避免对域名再做一次解析。
在mosdns中，我们随便添加几个国内DNS服务器和国外DNS服务器：
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/671a2e87-3b5e-41dd-b7d7-385ea064b809)
在高级选项中，我们修改并发数为3，同时像上游发起3个请求，我们用最快的那个，因为公司的网被限制了连接数，存在有时候得不到正确的DNS的时候，所以暴力发包是很有必要的。同时勾选防止DNS泄露，这个就是命中不了匹配域名的时候强制使用远程服务器查询。
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/96958231-e024-4697-8da5-592eea19532b)
点击保存并应用，这时候你可以看到系统默认的Dnsmasq被添加了一条转发规则，将dns请求交由mosdns来接管。同时因为shellcrash不会再劫持请求，等于所有的dns都由mosdns来发起。
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/73f5e40e-450b-47a1-ae71-25810509bd8f)
[最后访问](ipleak.net)查看效果，已经没有国内的DNS服务器响应了
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/02dc5cf2-6023-4e51-9ac5-3d895c007670)
附上一些链接：luci-app-mosdns[安装链接](https://github.com/IrineSistiana/mosdns/discussions/455)
我的路由器只有38M可用空间了，脚本检查要40M，修改下变量也能装。

PS:还是启用下crash的dns劫持比较好，不然域名分流规则会失效


