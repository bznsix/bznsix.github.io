在openwrt上运行了clash服务，用于接管下面所有设备的全局代理服务。但是发现移动宽带会对hysteria2协议进行一段时间的阻断。这时候所有的udp报文都会不通。但是可以重新拨号来绕过封禁。

编写脚本
```
#!/bin/bash

# 代理地址和端口
PROXY="127.0.0.1:7890"

# 日志文件路径
LOG_FILE="/tmp/check.log"

# 获取当前时间
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# 尝试通过代理访问Google首页
STATUS_CODE=$(curl --proxy "http://$PROXY" -s -o /dev/null -w "%{http_code}" https://www.google.com)

# 检查curl的返回值
if [ $? -ne 0 ] || [ "$STATUS_CODE" != "200" ]; then
    echo "$TIMESTAMP - 无法通过代理访问Google，返回状态码: $STATUS_CODE，重启WAN口..." >> "$LOG_FILE"
    # 重启WAN口
    /sbin/ifup wan >> "$LOG_FILE" 2>&1
else
    echo "$TIMESTAMP - 访问Google成功，状态码: $STATUS_CODE" >> "$LOG_FILE"
fi
```
每隔5分钟使用本机代理服务去访问google，如果失败就重启wan服务重新拨号。

可以看到日志：

![Image](https://github.com/user-attachments/assets/01296c53-8e8b-41ec-ac96-dda2caf7abdc)

确实检测到了一定时间的udp封禁

最后将脚本注册到crontab中，5min执行一次
```
*/5 * * * * /root/check_vpn.sh
```
完成！