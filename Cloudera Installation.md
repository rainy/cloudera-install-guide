<!-- CSS work goes here for the time being -->
<!-- set a:link text-decoration to none -->
<!-- set a:hover text-decoration to underline -->
<!-- http://forums.markdownpad.com/discussion/143/include-pdf-pagebreak-instructions-in-markdown/p1 -->

---

# <center> Introductions & Overview </center>
<strong> We will set up the cluster using Cloudera Manager </strong> <br>
<strong> Note: </strong> We need a 64-bit machine for Cloudera cluster set up.
* <a href="#intro_1"/> Selecting Hardware for Your CDH Cluster
* <a href="#intro_2"/> Linux configuration/prechecks
* <a href="#linux_config_lab"/> Install package repositories for MySQL, Cloudera Manager and CDH
* <a href="#linux_config_lab"/> Install a MySQL server for CM
* <a href="#linux_config_lab"/> Install Cloudera Manager
* <a href="#linux_config_lab"/> Install CDH
* <a href="#linux_config_lab"/> Testing
* <a href="#linux_config_lab"/> Kerberize the cluster

## <center> <a name="intro_1"/> Selecting Hardware for Your CDH Cluster
**Here are the recommended specifications for DataNode/TaskTrackers in a balanced Hadoop cluster:**
* 12-24 1-4TB hard disks in a JBOD (Just a Bunch Of Disks) configuration
* 2 quad-/hex-/octo-core CPUs, running at least 2-2.5GHz
* 64-512GB of RAM
* Bonded Gigabit Ethernet or 10Gigabit Ethernet (the more storage density, the higher the network throughput needed)

**Here are the recommended specifications for NameNode/JobTracker/Standby NameNode nodes. The drive count will fluctuate depending on the amount of redundancy:**
* 4–6 1TB hard disks in a JBOD configuration (1 for the OS, 2 for the FS image [RAID 1], 1 for Apache ZooKeeper, and 1 for Journal node)
* 2 quad-/hex-/octo-core CPUs, running at least 2-2.5GHz
* 64-128GB of RAM
* Bonded Gigabit Ethernet or 10Gigabit Ethernet

## <center> <a name="intro_2"/> Linux configuration/prechecks
**Before cluster set up, we need to configure our nodes. Follow the below steps in all nodes.** <br>

**Step 1:** Hostname Resolution, DNS and FQDNs <br>
* Set hostname
```bash
# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=ip-172-31-250-81.cn-north-1.compute.internal
NETWORKING_IPV6=no
NOZEROCONF=yes
```

* If you do use /etc/hosts, ensure that you are listing them in the appropriate order. <br>
    * The [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) must be listed first  
    * The IP address <code>127.0.0.1</code> must resolve to <code>localhost</code><p>
```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.31.250.81 ip-172-31-250-81.cn-north-1.compute.internal ip-172-31-250-81
172.31.250.119 ip-172-31-250-119.cn-north-1.compute.internal ip-172-31-250-119
172.31.250.120 ip-172-31-250-120.cn-north-1.compute.internal ip-172-31-250-120
172.31.250.121 ip-172-31-250-121.cn-north-1.compute.internal ip-172-31-250-121
```

* Test proper resolution <br>
```bash
# python -c 'import socket; print socket.getfqdn(), socket.gethostbyname(socket.getfqdn())'
```

* Enable the name server cache daemon (**nscd**) service <br>

**Step 2:** Sync all the nodes with a time source using NTP (Network Time Protocol) <br>
* follow the steps [documented here](http://www.cloudera.com/content/cloudera/en/documentation/core/v5-3-x/topics/cm_ig_mysql.html?scroll=cmig_topic_5_5#cmig_topic_5_5_1_unique_1).

**Step 3:** Make one user as sudo user, to be used later for SSH Ex: hypers <br>
```bash
#vim /etc/sudoers
#Add the below line:
dummyuser ALL=(ALL) NOPASSWD:ALL
```

**Step 4:** Set IPTables to off <br>
```bash
# /etc/init.d/iptables save
# /etc/init.d/iptables stop
# chkconfig iptables off
```

**Step 5:** Set IPv6 to disabled <br>
```bash
# vim /etc/sysctl.conf
#Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

# vim /etc/sysconfig/network-scripts/ifcfg-eth0
NETWORKING_IPV6=no
IPV6INIT=no
```

**Step 6:** Set SELinux to disabled <br>
```bash
# setenforce 0
# sed -i s@enforcing@disabled@g /etc/selinux/config
```

**Step 7:** Set swappiness (vm.swappiness) to 0 <br>
```bash
# sysctl vm.swappiness=0
# echo "vm_swappiness = 0" >> /etc/sysctl.conf
```

**Step 8:** Set Transparent Huge Pages (THP) to off <br>
```bash
# echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
# echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
# echo never > /sys/kernel/mm/transparent_hugepage/enabled
# echo never > /sys/kernel/mm/transparent_hugepage/defrag
# echo "echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag" >> /etc/rc.local
# echo "echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled" >> /etc/rc.local
# echo "echo never > /sys/kernel/mm/transparent_hugepage/enabled" >> /etc/rc.local
# echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag" >> /etc/rc.local
```

**Step 8:** Raise the global limits to 64k <br>
```bash
# vim /etc/security/limits.conf
#Add the below lines:
*   soft    nofile 655350
*   hard    nofile 655350 
```

**Step 9:** (Optional) If you are doing a lot of streaming, set vm.overcommit_memory kernel parameter to 1 <br>
```bash
# sysctl vm.overcommit_memory=1
# echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf
```

**Step 10:** Restart the network <br>
```bash
# /etc/init.d/network restart
```
