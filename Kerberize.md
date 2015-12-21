# <center> Introductions & Overview </center>
If you want to know more about Kerberos. Check out this [**link**](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Managing_Smart_Cards/Using_Kerberos.html)<br>
* <a href="#intro_1"/> Pre-requisite
* <a href="#intro_2"/> KDC Installation and Configuration
* <a href="#intro_3"/> Enabling Kerberos Authentication Using the Wizard
* <a href="#intro_4"/> Create the HDFS Superuser
* <a href="#intro_5"/> Create a Kerberos Principal and prepare the cluster for Each User Account
* <a href="#intro_6"/> Verify that Kerberos Security is Working


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


## <center> <a name="intro_3"/> Enabling Kerberos Authentication Using the Wizard
**Step 1:** Create a Kerberos Principal for the Cloudera Manager Server<br>
<code># kadmin.local</code><br>
<code># kadmin.local: addprinc -pw <Password> ***cloudera-scm/admin@SEBC.SIN***</code><p>

**Step 2:** Copy **/etc/krb5.conf** to each cluster node, including both service node and gateway node<br>

**Step 3:** If you are using **AES-256** Encryption, please install the JCE policy file for each host. There are 2 ways to do this:<br>
* In the Cloudera Manager Admin Console, navigate to the **Hosts** page. Both, the **Add New Hosts to Cluster** wizard and the **Re-run Upgrade Wizard** will give you the option to have Cloudera Manager install the JCE Policy file for you.
* You can follow the JCE Policy File installation instructions in the README.txt file included in the jce_policy-x.zip file.

**Step 4:** (Optional) To verify the type of encryption used in your cluster<br>
* On the local KDC host, type this command in the kadmin.local or kadmin shell to create a test principal:
<code>kadmin:  addprinc test</code><br>
* On a cluster host, type this command to start a Kerberos session as test:
<code># kinit test </code><br>
* On a cluster host, type this command to view the encryption type in use:
<code># klist -e </code><br>
  * If AES is being used, output like the following is displayed after you type the klist command (note that AES-256 is included in the output):
```bash
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: test@Cloudera Manager
Valid starting     Expires            Service principal
05/19/15 13:25:04  05/20/15 13:25:04  krbtgt/Cloudera Manager@Cloudera Manager
    Etype (skey, tkt): AES-256 CTS mode with 96-bit SHA-1 HMAC, AES-256 CTS mode with 96-bit SHA-1 HMAC 
```

**Step 5:** Properly configure Kerberos setting, navigate to "**Administration**" -> "**Security**" -> "**Enable Kerberos**"<br>
* KDC Type: MIT KDC
* KDC Server Host: **the FQDN of the KDC server**
* Kerberos Security Realm: **SEBC.SIN**
* Kerberos Encryption Type

**Step 6:** Do **NOT** use "**Manage krb5.conf through Cloudera Manager**" when asked. Click **Continue**<br>

**Step 7:** Import KDC Account Manager Credentials<br>
* Enter the **username** and **password** for the user that can create principals for CDH cluster in the KDC. This is the user/principal you created in **Step 1**: Create a Kerberos Principal for the Cloudera Manager Server

**Step 8:** (Optional) Configuring Custom Kerberos Principals<br>

**Step 9:** Configure HDFS DataNode Ports<br>
* Use the checkbox to confirm you are ready to restart the cluster. Click **Continue**

**Step 10:** Enabling Kerberos<br>
* This page lets you track the progress made by the wizard as it first stops all services on your cluster, deploys the krb5.conf, generates keytabs for other CDH services, deploys client configuration and finally restarts all services. Click **Continue**

**Step 8:** **Congratulations**<br>
* The final page lists the cluster(s) for which Kerberos has been successfully enabled. Click **Finish** to return to the Cloudera Manager Admin Console home page


## <center> <a name="intro_4"/> Create the HDFS Superuser
**Step 1:** In the kadmin.local or kadmin shell, type the following command to create a Kerberos principal called hdfs<br>
<code>kadmin:  addprinc hdfs@YOUR-LOCAL-REALM.COM</code><br>
  * **Note:** This command prompts you to create a password for the hdfs principal. You should use a strong password because having access to this principal provides superuser access to all of the files in HDFS.

**Step 2:** To run commands as the HDFS superuser, you must obtain Kerberos credentials for the hdfs principal. To do so, run the following command and provide the appropriate password when prompted<br>
<code># kinit hdfs@YOUR-LOCAL-REALM.COM</code><p>


## <center> <a name="intro_5"/> Create a Kerberos Principal and prepare the cluster for Each User Account
**Step 1:** In the kadmin.local or kadmin shell, use the following command to create user principals by replacing YOUR-LOCAL-REALM.COM with the name of your realm, and replacing USERNAME with a username:<br>
```bash 
kadmin:  addprinc USERNAME@YOUR-LOCAL-REALM.COM 

// Enter and re-enter a password when prompted
``` 
OR <br>
```bash 
# kadmin.local -q "addprinc -randkey rainy"
# kadmin.local -q "xst -norandkey -k rainy.keytab rainy@SEBC.SIN"
# kinit -k -t rainy.keytab rainy
# klist -e

If you want to destroy the Kerberos ticket, please type "kdestroy"
```

**Step 2:** Create user directory under */user* on HDFS for each user account. Change the owner and group of that directory to be the user. First you should login as *hdfs* user:<br>
<code># kinit hdfs</code><br>
<code># hadoop fs -mkdir /user/rainy</code><br>
<code># hadoop fs -chown rainy /user/rainy</code><p>


## <center> <a name="intro_6"/> Verify that Kerberos Security is Working
**Step 1:** (Optional) If Linux box is already integrated with AD (or other LDAP services) to do authentication, please ignore this step. Make sure all the hosts in the cluster have a Unix user account with same name as the first component of the user's principal name. Also to allow users submitting jobs, please look at the MapReduce configuration for **banned.users** and **min.user.id**, usually **banned.users** is set to mapred, hdfs and bin to prevent jobs from being submitted via those user accounts. And the default setting for the **min.user.id** property is 1000 to prevent jobs from being submitted with a user ID less than 1000. You can make changes to those two configurations if necessary.<br>

**Step 2:** Acquire Kerberos credentials for your user account.<br>
```bash 
# kinit USERNAME@YOUR-LOCAL-REALM.COM
``` 

**Step 3:** Enter a password when prompted.<br>

**Step 4:** Submit a sample pi calculation as a test MapReduce job. Use the following command if you use a package-based setup for Cloudera Manager:<br>
```bash 
# hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-0.20-mapreduce/hadoop-examples.jar pi 10 10000
Number of Maps = 10
Samples per Map = 10000
...
Job Finished in 38.572 seconds
Estimated value of Pi is 3.14120000000000000000
``` 

**Step 5:** You have now verified that Kerberos security is working on your cluster.










