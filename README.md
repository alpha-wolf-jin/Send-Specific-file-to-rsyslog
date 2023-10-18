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
[root@kafka git]# ip a | grep -w inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.122.33/24 brd 192.168.122.255 scope global noprefixroute enp1s0

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

# Append 2 line to /etc/rsyslog.conf

```
[root@aap-eda ~]# dnf install -y rsyslog

[root@aap-eda ~]# vim /etc/rsyslog.conf
...
action(type="omfwd" Target="192.168.122.33" Port="514" Protocol="udp")
action(type="omfwd" Target="192.168.122.33" Port="514" Protocol="tcp")

[root@aap-eda ~]# cat /etc/rsyslog.d/aap.conf 
$ModLoad imfile
input(type="imfile" File="/var/log/tower/tower.log" Tag="tower-log" Severity="info")
input(type="imfile" File="/var/log/tower/job_lifecycle.log" Tag="job_lifecycle-log" Severity="info")

[root@aap-eda log]# systemctl restart rsyslog

[root@aap-eda log]# systemctl status rsyslog
● rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2023-10-18 22:20:09 +08; 19min ago
     Docs: man:rsyslogd(8)
           https://www.rsyslog.com/doc/
 Main PID: 9725 (rsyslogd)
    Tasks: 4 (limit: 98188)
   Memory: 3.2M
   CGroup: /system.slice/rsyslog.service
           └─9725 /usr/sbin/rsyslogd -n

Oct 18 22:20:09 aap-eda.example.com systemd[1]: Starting System Logging Service...
Oct 18 22:20:09 aap-eda.example.com rsyslogd[9725]: [origin software="rsyslogd" swVersion="8.2102.0-13.el8" x-pid="9725" x-info="https://www.rsyslog.com"] start
Oct 18 22:20:09 aap-eda.example.com systemd[1]: Started System Logging Service.
Oct 18 22:20:09 aap-eda.example.com rsyslogd[9725]: imjournal: journal files changed, reloading...  [v8.2102.0-13.el8 try https://www.rsyslog.com/e/0 ]

```

# Verify the central log

**Confirm if the logs are sent to the central message file on the remote rsyslog server.**

```
root@kafka git]# grep tower /var/log/messages
Oct 18 22:11:05 aap-eda tower-log 2023-10-18 14:11:01,866 INFO     [06a447931ec2408a901620f5ebc6599a] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 731
Oct 18 22:11:05 aap-eda tower-log 2023-10-18 14:11:01,866 INFO     [06a447931ec2408a901620f5ebc6599a] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 731
Oct 18 22:18:28 aap-eda tower-log 2023-10-18 14:18:25,156 INFO     [7ce7f5a946fb403193d0703bcacc6a39] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 732
Oct 18 22:18:28 aap-eda tower-log 2023-10-18 14:18:25,156 INFO     [7ce7f5a946fb403193d0703bcacc6a39] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 732
Oct 18 22:20:39 aap-eda tower-log 2023-10-18 14:20:32,190 INFO     [76a76e595efa4218a4b0482f84d99f72] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 733
Oct 18 22:20:39 aap-eda tower-log 2023-10-18 14:20:32,190 INFO     [76a76e595efa4218a4b0482f84d99f72] awx.main.commands.run_callback_receiver Starting EOF event processing for Job 733

[root@kafka log]# grep job_lifecycle-log messages
Oct 18 22:17:38 aap-eda job_lifecycle-log {"type": "job", "task_id": 712, "state": "created", "work_unit_id": null, "task_name": "recommendation", "guid": "1af8da004b464c758bd5b6cbe7551d13", "time": "2023-10-18T03:42:46.311515+00:00"}
Oct 18 22:17:38 aap-eda job_lifecycle-log {"type": "job", "task_id": 712, "state": "controller_node_chosen", "work_unit_id": null, "task_name": "recommendation", "controller_node": "aap-eda.example.com", "guid": "1af8da004b464c758bd5b6cbe7551d13", "time": "2023-10-18T03:42:46.454136+00:00"}
Oct 18 22:17:38 aap-eda job_lifecycle-log {"type": "job", "task_id": 712, "state": "execution_node_chosen", "work_unit_id": null, "task_name": "recommendation", "execution_node": "aap-eda.example.com", "guid": "1af8da004b464c758bd5b6cbe7551d13", "time": "2023-10-18T03:42:46.454442+00:00"}
Oct 18 22:17:38 aap-eda job_lifecycle-log {"type": "job", "task_id": 712, "state": "waiting", "work_unit_id": null, "task_name": "recommendation", "guid": "1af8da004b464c758bd5b6cbe7551d13", "time": "2023-10-18T03:42:46.508027+00:00"}


```
