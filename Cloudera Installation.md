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
* <a href="#intro_3"/> Install package repositories for Cloudera Manager and CDH
* <a href="#intro_4"/> Install a MySQL server for CM
* <a href="#intro_5"/> Install Cloudera Manager and CDH
* <a href="#intro_6"/> Benchmarking
* <a href="#intro_7"/> Kerberize the cluster


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
* follow the steps [documented here](https://github.com/rainy/cloudera-install-guide/blob/master/NTP.md).

**Step 3:** Make one user as sudo user, to be used later for SSH Ex: hypers <br>
```bash
# vim /etc/sudoers
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

**Step 9:** Set noatime on your supplementary volumes <br>
* Set it via mount option in **/etc/fstab**
```bash
# vim /etc/fstab
/dev/sdb1 /data1    ext4    defaults,noatime       0 0
```

**Step 10:** Set the reserve space for your supplementary volumes to 0 <br>
* Set it via mount option in **/etc/fstab**
```bash
# vim /etc/fstab
/dev/sdb1 /data1    ext4    defaults,noatime       0 0
```

**Step 11: (Optional)** If you are doing a lot of streaming, set vm.overcommit_memory kernel parameter to 1 <br>
```bash
# sysctl vm.overcommit_memory=1
# echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf
```

**Step 12:** Restart the network <br>
```bash
# /etc/init.d/network restart
```


## <center> <a name="intro_3"/> Install package repositories for Cloudera Manager and CDH
**Step 1:** Installing and Starting Apache HTTPD <br>
```bash
# yum install httpd -y
# chkconfig httpd on
# service httpd start
```

**Step 2:** Download Tarball of CM <br>
```bash
# cd /var/www/html/
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/CDH-5.5.0/cm5.5.0-centos6.tar.gz
# tar xzvf cm5.5.0-centos6.tar.gz
# chmod -R ugo+rX /var/www/html/cm
# rm -rf cm5.5.0-centos6.tar.gz
```

**Step 3:** Download Parcel of CDH <br>
```bash
# cd /var/www/html/
# mkdir CDH-5.5.0
# cd CDH-5.5.0/
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/CDH-5.5.0/CDH-5.5.0-1.cdh5.5.0.p0.8-el6.parcel
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/CDH-5.5.0/CDH-5.5.0-1.cdh5.5.0.p0.8-el6.parcel.sha1
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/CDH-5.5.0/manifest.json
```

**Step 4:** Download Parcel of KAFKA <br>
```bash
# cd /var/www/html/
# mkdir KAFKA
# cd KAFKA/
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/Kafka/KAFKA-0.8.2.0-1.kafka1.3.2.p0.15-el6.parcel
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/Kafka/KAFKA-0.8.2.0-1.kafka1.3.2.p0.15-el6.parcel.sha1
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/Kafka/manifest.json
```

**Step 5:** Enable all nodes to find the packages that you are hosting (Follow the below steps in all nodes) <br>
* replace **ip-172-31-250-81.cn-north-1.compute.internal** with your local repository's hostname
```bash
echo "[cloudera-manager]" > /etc/yum.repos.d/cloudera-manager.repo
echo "# Packages for Cloudera Manager, Version 5, on RedHat or CentOS 6 x86_64" >> /etc/yum.repos.d/cloudera-manager.repo
echo "name=Cloudera Manager" >> /etc/yum.repos.d/cloudera-manager.repo
echo "baseurl = http://ip-172-31-250-81.cn-north-1.compute.internal/cm/5/" >> /etc/yum.repos.d/cloudera-manager.repo
echo "gpgkey = http://ip-172-31-250-81.cn-north-1.compute.internal/cm/RPM-GPG-KEY-cloudera" >> /etc/yum.repos.d/cloudera-manager.repo
echo "gpgcheck = 1" >> /etc/yum.repos.d/cloudera-manager.repo
```


## <center> <a name="intro_4"/> Install a MySQL server for CM
* follow the steps [documented here](https://github.com/rainy/cloudera-install-guide/blob/master/MySQL.md)


## <center> <a name="intro_5"/> Install Cloudera Manager and CDH
**Step 1:** Set up a Database for the Cloudera Manager Server <br>
```bash
mysql -uroot --password='gurutechhypers' -h cdh01.hypers.com.cn
    GRANT ALL PRIVILEGES ON scm.* to 'scm'@'%' IDENTIFIED BY 'xRoYuK8ajV';
    flush privileges;
    exit;
```

**Step 2:** Set up an external database and pre-create the schemas needed for your deployment <br>
```bash
mysql> create database database DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on database.* TO 'user'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.00 sec)
```
* **database**, **user**, and **password** can be any value. The examples match the default names provided in the Cloudera Manager configuration settings:<br>

| Role | Database | User | Password |
| ------------ | ------------ | ------------ | ------------ |
| Activity Monitor | amon | amon | amon_password |
| Reports Manager | rman | rman | rman_password |
| Hive Metastore Server | metastore | hive | hive_password |
| Sentry Server | sentry | sentry | sentry_password |
| Cloudera Navigator Audit Server | nav | nav | nav_password |
| Cloudera Navigator Metadata Server | navms | navms | navms_password |

**Step 3:** Install the Oracle JDK <br>
* Install the Oracle Java Development Kit (JDK) on the Cloudera Manager Server host <br>
```bash
# yum install oracle-j2sdk1.7 -y
```

**Step 4:** Install the Cloudera Manager Server Packages <br>
* On the Cloudera Manager Server host, type the following commands to install the Cloudera Manager packages <br>
```bash
# yum install cloudera-manager-daemons cloudera-manager-server -y
```

**Step 4:** Set up a Database for the Cloudera Manager Server <br>
* Running the script when MySQL is installed on another host
* This example explains how to run the script on the Cloudera Manager Server host (myhost2) and create and use a temporary MySQL user account to connect to MySQL remotely on the MySQL host (myhost1) 
* On the Cloudera Manager Server host (myhost2), run the script<br>
<code>/usr/share/cmf/schema/scm_prepare_database.sh mysql -h *cdh01.hypers.com.cn* -u*root* -p*gurutechhypers* --scm-host *cdh01.hypers.com.cn* *scm* *scm* *xRoYuK8ajV*</code><p>

**Step 5:** Start the Cloudera Manager Server <br>
```bash
# chkconfig cloudera-scm-server on
# service cloudera-scm-server start
```

**Step 6:** Start and Log into the Cloudera Manager Admin Console <br>
* In a web browser, enter **http://Server host:7180**
* Log into Cloudera Manager Admin Console. The default credentials are: **Username:** admin **Password:** admin
* After logging in, the **Cloudera Manager End User License Terms and Conditions** page displays. Read the terms and conditions and then select **Yes** to accept them
* Click **Continue**

**Step 7:** Choose Cloudera Manager Edition and Hosts 
* When you start the Cloudera Manager Admin Console, the install wizard starts up. Click **Continue** to get started
* Choose which **edition** to install
* (Optional) If you elect Cloudera Enterprise, install a license
* Click **Continue** to proceed with the installation
* Enter the cluster hostnames or IP addresses. You can also specify hostname and IP address ranges. Click **Search**
* Click **Continue**. The Select Repository screen displays

**Step 8:** Choose the Software Installation Type and Install Software 
* Parcel Repository - In the **Remote Parcel Repository URLs** field, click the **+** button and enter the URL of the repository
    * replace **ip-172-31-250-81.cn-north-1.compute.internal** with your local repository's hostname
    * <code>http://ip-172-31-250-81.cn-north-1.compute.internal/CDH-5.5.0</code>
    * <code>http://ip-172-31-250-81.cn-north-1.compute.internal/KAFKA</code>
* Select the release of Cloudera Manager Agent. You can choose either the version that matches the Cloudera Manager Server you are currently using or specify a version in a custom repository. If you opted to use custom repositories for installation files, you can provide a GPG key URL that applies for all repositories. Click **Continue**
    * replace **ip-172-31-250-81.cn-north-1.compute.internal** with your local repository's hostname
    * <code>http://ip-172-31-250-81.cn-north-1.compute.internal/cm/5/</code>
    * <code>http://ip-172-31-250-81.cn-north-1.compute.internal/cm/RPM-GPG-KEY-cloudera</code>
* Select the **Install Oracle Java SE Development Kit (JDK)** checkbox to allow Cloudera Manager to install the JDK on each cluster host or leave deselected if you installed it. If checked, your local laws permit you to deploy unlimited strength encryption, and you are running a secure cluster, select the **Install Java Unlimited Strength Encryption Policy Files** checkbox. Click **Continue**
* Do **NOT** use **single user mode** when asked. Click **Continue**
* If you chose to have Cloudera Manager install software, specify host installation properties
    * Select **root** or enter the user name for an account that has password-less sudo permission
    * Select an authentication method
* Click **Continue**. When the **Continue** button at the bottom of the screen turns **blue**, the installation process is completed
* Click **Continue**. The Host Inspector runs to validate the installation and provides a summary of what it finds, including all the versions of the installed components. If the validation is successful, click **Finish**

**Step 9:** Add Services 
* In the first page of the Add Services wizard, choose the combination of services to install and whether to install Cloudera Navigator. Click **Continue**
* Customize the assignment of role instances to hosts. When you are satisfied with the assignments, click **Continue**
* On the Database Setup page, configure settings for required databases. Click **Test Connection** to confirm that Cloudera Manager can communicate with the database using the information you have supplied. If the test succeeds in all cases, click **Continue**
* Review the configuration changes to be applied. Confirm the settings entered for file system paths. Click **Continue**. The wizard starts the services
* When all of the services are started, click **Continue**
* Click **Finish** to proceed to the **Cloudera Manager Admin Console Home Page**

**Step 10:** Change the Default Administrator Password 
* Right-click the logged-in username at the far right of the top navigation bar and select **Change Password**
* Enter the current password and a new password twice, and then click **Update**


## <center> <a name="intro_6"/> Benchmarking <br>
**[Michael G. Noll's blog post](http://www.michael-noll.com/blog/2011/04/09/benchmarking-and-stress-testing-an-hadoop-cluster-with-terasort-testdfsio-nnbench-mrbench/) reviews many of benchmark tools** <br>
**Step 1:** Running a MapReduce Job <br>
* **Parcel** - <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100</code>
* **Package** - <code>sudo -u hdfs hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100</code>

**Step 2:** TeraSort benchmark suite <br>
* <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar teragen 1000000 /user/hdfs/terasort-input</code>
* <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar terasort /user/hdfs/terasort-input /user/hdfs/terasort-output</code>
* <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar teravalidate /user/hdfs/terasort-output /user/hdfs/terasort-validate</code>

**Step 3:** NameNode benchmark (nnbench) <br>
* <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-0.20-mapreduce/hadoop-test-2.6.0-mr1-cdh5.5.0.jar nnbench -operation create_write -maps 12 -reduces 6 -blockSize 1 -bytesToWrite 0 -numberOfFiles 1000 -replicationFactorPerFile 3 -readFileAfterOpen true -baseDir /benchmarks/NNBench</code>



## <center> <a name="intro_7"/> Kerberize the cluster <br>
* Document of Cloudera Manager integrate MIT Kerberos [documented here](https://github.com/rainy/cloudera-install-guide/blob/master/Kerberize.md)
* Document of Cloudera Manager integrate FreeIPA by Smoak.Wu of Hypers [documented here](https://git.hypers.com/OP/cloudera-krb)

## <center> <a name="intro_7"/> Validate <br>
**MapReduce:** <br>
* <code>sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100</code>
**Spark:** <br>
* <code>spark-submit --num-executors 5 --master yarn-cluster --class org.apache.spark.examples.SparkPi  /opt/cloudera/parcels/CDH/jars/spark-examples-1.6.0-cdh5.7.1-hadoop2.6.0-cdh5.7.1.jar 100</code>






