## 摘要
使用脚本对整机进行重启测试。
## 测试项说明
测试机器重启会不会出现死机等问题。
## 测试计划
进行重启测试 500 次。
## 操作步骤
在 /lib/system/system 下创建名为 cycletest.service_ 重启脚本：

```
[Unit]
Description=Reboots unit after 30s

[Service]
StandardOutput=syslog+console
ExecStart=/bin/sh -c "\
test -f /cycle-count || echo 0 > /cycle-count;\
echo 'starting cycletest';\
sleep 30;\
expr `cat /cycle-count` + 1 > /cycle-count;\
systemctl reboot;\
"

[Install]
WantedBy=multi-user.target
```

后根据以下指令安装并开始测试：

```
systemctl daemon-reload
systemctl enable cycletest.service (enable the service to start on reboot)
systemctl start cycletest.service (start the service, should reboot in 30s)
```
