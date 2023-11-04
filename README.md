# sing-box

The universal proxy platform.

***

### 1. ProxyProvider 支持

- 编译时需要使用 `with_proxyprovider` tag

##### 配置详解
```json5
{
  "proxyproviders": [
    {
      "tag": "proxy-provider-x", // 标签，必填，用于区别不同的 proxy-provider，不可重复，设置后outbounds会暴露一个同名的selector出站
      "url": "", // 订阅链接，必填，支持Clash订阅链接，支持普通分享链接，支持Sing-box订阅链接
      "cache_file": "/tmp/proxy-provider-x.cache", // 缓存文件，选填，强烈建议填写，可以加快启动速度
      "update_interval": "4h", // 更新间隔，选填，仅填写 cache_file 有效，若当前缓存文件已经超过该时间，将会进行后台自动更新
      "request_timeout": "10s", // 请求超时时间
      "use_h3": false, // 使用 HTTP/3 请求订阅
      "dns": "tls://223.5.5.5", // 使用自定义 DNS 请求订阅域名
      "tag_format": "proxy-provider - %s", // 如果有多个订阅并且订阅间存在重名节点，可以尝试使用，其中 %s 为占位符，会被替换为原节点名。比如：原节点名："HongKong 01"，tag_format设置为 "PP - %s"，替换后新节点名会更变为 "PP - HongKong 01"，以解决节点名冲突的问题
      "global_filter": {
        "white_mode": true, // 白名单模式，匹配的节点会被保留，不匹配的节点会被删除
        "rules": [], // 规则，详情见下文
      },
      // 规则
      // 1. Golang 正则表达式 (example: Node) ==> 匹配 Tag (匹配 Node)
      // 2. tag:Golang 正则表达式 (example: tag:Node) ==> 匹配 Tag (匹配 Node)
      // 3. type:Golang 正则表达式 (example: type:vmess) ==> 匹配 Type (节点类型) (匹配 vmess)
      // 4. server:Golang 正则表达式 (example: server:1.1.1.1) ==> 匹配 Server (节点服务器地址，不含端口) (匹配 1.1.1.1)
      // 5. 若设置 tag_format 则匹配的是替换前的节点名
      "lookup_ip": false, // 是否查询 IP 地址，覆盖节点地址，需要设置 dns 字段
      "download_ua": "clash.meta", // 更新订阅时使用的 User-Agent
      "dialer": {}, // 附加在节点 outbound 配置的 Dial 字段
      "request_dialer": {}, // 请求时使用的 Dial 字段配置，detour 字段无效
      "running_detour": "", // 运行时后台自动更新所使用的 outbound
      "groups": [ // 自定义分组
        {
          "tag": "", // outbound tag，必填
          "type": "selector", // outbound 类型，必填，仅支持selector, urltest
          "filter": {}, // 节点过滤规则，选填，详见上global_filter字段
          ... Selector 或 URLTest 其他字段配置
        }
      ]
    }
  ]
}
```

##### DNS 支持格式
```
tcp://1.1.1.1
tcp://1.1.1.1:53
tcp://[2606:4700:4700::1111]
tcp://[2606:4700:4700::1111]:53
udp://1.1.1.1
udp://1.1.1.1:53
udp://[2606:4700:4700::1111]
udp://[2606:4700:4700::1111]:53
tls://1.1.1.1
tls://1.1.1.1:853
tls://[2606:4700:4700::1111]
tls://[2606:4700:4700::1111]:853
tls://1.1.1.1/?sni=cloudflare-dns.com
tls://1.1.1.1:853/?sni=cloudflare-dns.com
tls://[2606:4700:4700::1111]/?sni=cloudflare-dns.com
tls://[2606:4700:4700::1111]:853/?sni=cloudflare-dns.com
https://1.1.1.1
https://1.1.1.1:443/dns-query
https://[2606:4700:4700::1111]
https://[2606:4700:4700::1111]:443
https://1.1.1.1/dns-query?sni=cloudflare-dns.com
https://1.1.1.1:443/dns-query?sni=cloudflare-dns.com
https://[2606:4700:4700::1111]/dns-query?sni=cloudflare-dns.com
https://[2606:4700:4700::1111]:443/dns-query?sni=cloudflare-dns.com
1.1.1.1 => udp://1.1.1.1:53
1.1.1.1:53 => udp://1.1.1.1:53
[2606:4700:4700::1111] => udp://[2606:4700:4700::1111]:53
[2606:4700:4700::1111]:53 => udp://[2606:4700:4700::1111]:53
```

##### 简易配置示例
```json5
{
  "proxyproviders": [
    {
      "tag": "标签",
      "url": "订阅链接",
      "cache_file": "缓存文件路径",
      "dns": "tcp://223.5.5.5",
      "update_interval": "4h", // 自动更新缓存
      "request_timeout": "10s" // 请求超时时间
    }
  ]
}
```

### 2. 为 RuleSet 适配 ClashAPI (Rule Provider)