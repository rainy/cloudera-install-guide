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
