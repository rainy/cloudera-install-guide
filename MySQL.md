**Step 1:** Download Tarball of CM <br>
```bash
# mkdir -p /var/www/html/mysql/packages
# cd /var/www/html/mysql/packages

// Mysql5.6
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-community-client-5.6.24-3.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-community-common-5.6.24-3.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-community-libs-5.6.24-3.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-community-libs-compat-5.6.24-3.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-community-server-5.6.24-3.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-connector-java-5.1.29-1.noarch.rpm
// Mysql5.5
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-community-client-5.5.46-2.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-community-common-5.5.46-2.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-community-libs-5.5.46-2.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-community-libs-compat-5.5.46-2.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-community-server-5.5.46-2.el6.x86_64.rpm
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql5.5/mysql-connector-java-5.1.29-1.noarch.rpm

# cd /var/www/html/mysql/
# yum install -y createrepo
# createrepo ./
```

**Step 2:** Enable all nodes to find the packages that you are hosting (Follow the below steps in all nodes) <br>
* replace **ip-172-31-250-81.cn-north-1.compute.internal** with your local repository's hostname
```bash
echo "[mysql-community]" > /etc/yum.repos.d/mysql.repo
echo "# Packages for MySQL, Version 5.5 or 5.6, on RedHat or CentOS 6 x86_64" >> /etc/yum.repos.d/mysql.repo
echo "name=Cloudera Manager" >> /etc/yum.repos.d/mysql.repo
echo "baseurl = http://ip-172-31-250-81.cn-north-1.compute.internal/mysql/" >> /etc/yum.repos.d/mysql.repo
echo "enabled = 1" >> /etc/yum.repos.d/mysql.repo
echo "gpgcheck = 0" >> /etc/yum.repos.d/mysql.repo
```

##Installing the MySQL Server
**Step 1:** Install the MySQL database <br>
```bash
# yum install mysql mysql-server -y
```

**Step 2:** Run the <code>mysql_install_db</code> as the <code>mysql</code> user on each node before starting the <code>mysqld</code> service. This will create files with correct permissions <br>
```bash
# sudo -u mysql mysql_install_db
```

**Step 3:** Edit your <code>/etc/my.cnf</code> **before** you start MySQL.  <br>
* Set the max_connections property according to the size of your cluster:
    *  Allow 100 maximum connections for each database and then add 50 extra connections
* **Configure MySQL with a replica server**
    * Comment out the line of <code>server-id</code> if without replica server
    * <code>server-id</code> with 1,2,3...
    * <code>auto-increment-increment</code> with 1,2,3...
```bash
[mysqld]
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
# symbolic-links = 0

server-id = 1
auto-increment-offset = 2
auto-increment-increment = 1

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Recommended in standard MySQL setup
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
sql_mode=STRICT_ALL_TABLES

#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system
#and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
binlog_format = mixed
log-bin=/var/lib/mysql/mysql-bin
relay-log=/var/lib/mysql/relay-mysql

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

# charset 
character-set-server  = utf8 
collation-server      = utf8_general_ci 
character_set_server  = utf8 
collation_server      = utf8_general_ci

skip-name-resolve 

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
# Default is Latin1, if you need UTF-8 set this (also in server section)
default-character-set = utf8
```

**Step 4:** Ensure the MySQL server starts at boot  <br>
```bash
# chkconfig mysqld on
# chkconfig --list mysqld
```

**Step 5:** Start the MySQL server  <br>
```bash
# service mysqld start
```

**Step 6:** Set the MySQL root password. In the following example, the current root password is blank. Press the **Enter** key when you're prompted for the root password  <br>
```bash
# /usr/bin/mysql_secure_installation
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
All done!
```

##Installing the MySQL JDBC Driver
* **Follow the below steps in all nodes**
```bash
# mkdir /usr/share/java/
# cd /usr/share/java/
# wget https://s3.cn-north-1.amazonaws.com.cn/hypers/cdh/mysql/mysql-connector-java-5.1.37-bin.jar
# ln -s mysql-connector-java-5.1.37-bin.jar mysql-connector-java.jar
```
