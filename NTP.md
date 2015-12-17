##NTP Server
* **0.rhel.pool.ntp.org** is the NTP synchronization source
```bash
echo "driftfile /var/lib/ntp/drift" > /etc/ntp.conf 
echo "#restrict default nomodify notrap nopeer noquery" >> /etc/ntp.conf
echo "restrict 127.0.0.1" >> /etc/ntp.conf
echo "restrict -6 ::1" >> /etc/ntp.conf
echo "restrict 172.31.250.0 mask 255.255.255.0 nomodify notrap" >> /etc/ntp.conf
echo "server 0.rhel.pool.ntp.org" >> /etc/ntp.conf
echo "server 127.127.1.0" >> /etc/ntp.conf
echo "fudge 127.127.1.0 stratum 10" >> /etc/ntp.conf
echo "includefile /etc/ntp/crypto/pw" >> /etc/ntp.conf
echo "keys /etc/ntp/keys" >> /etc/ntp.conf
echo "broadcastdelay 0.008" >> /etc/ntp.conf
cat /etc/ntp.conf
```

```bash
\cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ntpdate 0.rhel.pool.ntp.org
chkconfig ntpd on
service ntpd restart
```

##NTP Client
* replace **ip-172-31-250-81.cn-north-1.compute.internal** with your NTP Server's hostname
```bash
# vim /etc/ntp.conf
#Add the below line:
server ip-172-31-250-81.cn-north-1.compute.internal

\cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
chkconfig ntpd on
service ntpd restart
```
