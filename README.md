# Send-Specific-file-to-rsyslog

```
echo "# Send-Specific-file-to-rsyslog" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/alpha-wolf-jin/Send-Specific-file-to-rsyslog.git
git config --global credential.helper 'cache --timeout 720000'
git push -u origin main

git add . ; git commit -a -m "update README" ; git push -u origin main
```

**Reference**
https://www.casesup.com/category/knowledgebase/howtos/how-to-forward-specific-log-file-to-a-remote-syslog-server
https://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-setup-centralized-syslog-server-on-centos-8-rhel-8.html

# Remote Syslog Server Set Up

```
[root@kafka git]# yum update rsyslog -y

[root@kafka git]# vim /etc/rsyslog.conf
...
# Provides UDP syslog reception
# for parameters see http://www.rsyslog.com/doc/imudp.html
module(load="imudp") # needs to be done just once
input(type="imudp" port="514")

# Provides TCP syslog reception
# for parameters see http://www.rsyslog.com/doc/imtcp.html
module(load="imtcp") # needs to be done just once
input(type="imtcp" port="514")
...

[root@kafka git]# systemctl restart rsyslog

[root@kafka git]# systemctl status rsyslog
● rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2023-10-18 21:40:33 +08; 51min ago
     Docs: man:rsyslogd(8)
           https://www.rsyslog.com/doc/
 Main PID: 13493 (rsyslogd)
    Tasks: 9 (limit: 48231)
   Memory: 2.4M
   CGroup: /system.slice/rsyslog.service
           └─13493 /usr/sbin/rsyslogd -n

Oct 18 21:40:33 kafka.example.com systemd[1]: Starting System Logging Service...
Oct 18 21:40:33 kafka.example.com rsyslogd[13493]: [origin software="rsyslogd" swVersion="8.2102.0-13.el8" x-pid="13493" x-info="https://www.rsyslog.com"] start
Oct 18 21:40:33 kafka.example.com systemd[1]: Started System Logging Service.
Oct 18 21:40:33 kafka.example.com rsyslogd[13493]: imjournal: journal files changed, reloading...  [v8.2102.0-13.el8 try https://www.rsyslog.com/e/0 ]

```

# Syslog Client Server Set Up

Specify the log files to be sent to remote rsyslog


