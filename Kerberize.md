# <center> Introductions & Overview </center>
If you want to know more about Kerberos. Check out this [**link**](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Managing_Smart_Cards/Using_Kerberos.html)<br>
* <a href="#intro_1"/> Pre-requisite
* <a href="#intro_2"/> KDC Installation and Configuration
* <a href="#intro_3"/> Enable Kerberos in Your Cluster
* <a href="#intro_4"/> Test Kerberos Authentication


## <center> <a name="intro_1"/> Pre-requisite
* You have a Hadoop Cluster managed by Cloudera Manager. If you dont have one, check this [**link**](https://github.com/rainy/cloudera-install-guide/blob/master/Cloudera%20Installation.md) to create a cluster managed by cloudera manager
* You have the DNS server configured, and each host in the cluster is accessible via DNS<p>


## <center> <a name="intro_2"/> KDC Installation and Configuration
**Step 1:** To install packages for a Kerberos server <br>
<code># yum install -y krb5-server krb5-libs krb5-auth-dialog krb5-workstation</code><p>

**Step 2:** Modify the /etc/krb5.conf configuration <br>
<code># vim /etc/krb5.conf</code><br>
<code>[logging]</code><br>
<code> default = FILE:/var/log/krb5libs.log</code><br>
<code> kdc = FILE:/var/log/krb5kdc.log</code><br>
<code> admin_server = FILE:/var/log/kadmind.log</code><br>
<br>
<code>[libdefaults]</code><br>
<code> default_realm = ***SEBC.SIN***</code><br>
<code> dns_lookup_realm = false</code><br>
<code> dns_lookup_kdc = false</code><br>
<code> ticket_lifetime = 24h</code><br>
<code> renew_lifetime = 7d</code><br>
<code> forwardable = true</code><br>
<br>
<code>[realms]</code><br>
<code> ***SEBC.SIN*** = {</code><br>
<code>  kdc = ***ip-172-31-250-81.cn-north-1.compute.internal***</code><br>
<code>  admin_server = ***ip-172-31-250-81.cn-north-1.compute.internal***</code><br>
<code> }</code><br>
<br>
<code>[domain_realm]</code><br>
<code> ***.cn-north-1.compute.internal*** = ***SEBC.SIN***</code><br>
<code> ***cn-north-1.compute.internal*** = ***SEBC.SIN***</code><p>

**Step 3:** Configure /var/kerberos/krb5kdc/kdc.conf<br>
<code>[kdcdefaults]</code><br>
<code> kdc_ports = 88</code><br>
<code> kdc_tcp_ports = 88</code><br>
<br>
<code>[realms]</code><br>
<code> ***SEBC.SIN*** = {</code><br>
<code>  #master_key_type = aes256-cts</code><br>
<code>  ***max_renewable_life = 7d 0h 0m 0s***</code><br>
<code>  acl_file = /var/kerberos/krb5kdc/kadm5.acl</code><br>
<code>  dict_file = /usr/share/dict/words</code><br>
<code>  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab</code><br>
<code>  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal</code><br>
<code> }</code><p>

**Step 4:** Configure /var/kerberos/krb5kdc/kadm5.acl<br>
<code>\*/admin@***SEBC.SIN***	\*</code><p>

**Step 5:** Create Kerberos Database<br>
<code># kdb5_util create -r ***SEBC.SIN*** -s</code><p>

**Step 6:** Create Kerberos Administrator Principal<br>
<code># kadmin.local</code><br>
<code># kadmin.local: addprinc ***admin/admin@SEBC.SIN***</code><p>

**Step 7:** Start the Kerberos Daemons on the KDC<br>
<code># chkconfig krb5kdc on </code><br>
<code># chkconfig kadmin on </code><br>
<code># service krb5kdc start</code><br>
<code># service kadmin start</code><p>

**Step 8:** Test the Kerberos<br>
<code># kinit ***admin/admin@SEBC.SIN***</code><br>
<code># klist -e</code><p>

**Step 9:** Install the Kerberos client and utilities on all other cluster nodes<br>
<code># yum install -y krb5-libs krb5-auth-dialog krb5-workstation</code><p>

**Step 10:** Install the OpenLDAP client on Cloudera Manager node<br>
<code># yum install -y openldap-clients</code><p>


## <center> <a name="intro_3"/> Enable Kerberos in Your Cluster
**Step 1:** Create Cloudera Manager Administrator Principal<br>
<code># kadmin.local</code><br>
<code># kadmin.local: addprinc ***cloudera-scm/admin@SEBC.SIN***</code><p>

**Step 2:** Copy **/etc/krb5.conf** to each cluster node, including both service node and gateway node<br>

**Step 3:** If you are using **AES-256** Encryption, please install the JCE policy file for each host. (You can re-run the upgrade wizard in host page, navigate to "**Hosts**" -> "**Re-run Upgrade Wizard**")<br>

**Step 4:** Properly configure Kerberos setting, navigate to "**Administration**" -> "**Security**" -> "**Enable Kerberos**"<br>
* KDC Type
* KDC Server Host: **the FQDN of the KDC server**
* Kerberos Security Realm: **SEBC.SIN**
* Kerberos Encryption Type

**Step 5:** Follow the Wizard<br>


## <center> <a name="intro_4"/> Test Kerberos Authentication
**Step 1:** (Optional) If Linux box is already integrated with AD (or other LDAP services) to do authentication, please ignore this step. Make sure all the hosts in the cluster have a Unix user account with same name as the first component of the user's principal name. Also to allow users submitting jobs, please look at the MapReduce configuration for **banned.users** and **min.user.id**, usually **banned.users** is set to mapred, hdfs and bin to prevent jobs from being submitted via those user accounts. And the default setting for the **min.user.id** property is 1000 to prevent jobs from being submitted with a user ID less than 1000. You can make changes to those two configurations if necessary.<br>

**Step 2:** Create user directory under */user* on HDFS for each user account. Change the owner and group of that directory to be the user. Assume you have the hdfs Keytab files like *hdfs.keytab*, so first you should login as *hdfs* user:<br>
<code># kinit hdfs</code> or <code># kinit -k -t hdfs.keytab hdfs</code><br>
<code># hadoop fs -mkdir /user/rainy</code><br>
<code># hadoop fs -chown rainy /user/rainy</code><p>

**Step 3:** Create the user principal, and authenticate the user<br>
* create user principal: <code># kadmin.local -q "addprinc -randkey rainy"</code><br>
* retrieve the keytab file: <code># kadmin.local -q "xst -norandkey -k rainy.keytab rainy@SEBC.SIN"</code><br>
* authenticate the user: <code># kinit -k -t rainy.keytab rainy</code><br>
* list the authenticated user: <code># klist</code><br>
* If you want to destroy the Kerberos ticket, please type "kdestroy"<p>












