# Summary

Use a script to conduct reboot tests on the entire machine.

## Test Item Description

Test if the machine will experience issues such as freezes during a reboot.

## Test Plan

Conduct a reboot test 500 times.

## Operational Steps

Create a reboot script named `cycletest.service` in `/lib/system/system`:

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

Then, install and begin the test using the following commands:

```
systemctl daemon-reload
systemctl enable cycletest.service # enable the service to start on reboot
systemctl start cycletest.service # start the service, machine should reboot in 30s
```
