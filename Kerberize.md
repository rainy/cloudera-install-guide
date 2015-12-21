# <center> Introductions & Overview </center>
If you want to know more about Kerberos. Check out this [**link**](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Managing_Smart_Cards/Using_Kerberos.html)<br>
* <a href="#intro_1"/> Pre-requisite
* <a href="#intro_2"/> KDC Installation and Configuration
* <a href="#intro_3"/> Configure MySQL with a replica server
* <a href="#intro_4"/> Installing the MySQL JDBC Driver


## <center> <a name="intro_1"/> Pre-requisite
* You have a Hadoop Cluster managed by Cloudera Manager. If you dont have one, check this [**link**](https://github.com/rainy/cloudera-install-guide/blob/master/Cloudera%20Installation.md) to create a cluster managed by cloudera manager
* You have the DNS server configured, and each host in the cluster is accessible via DNS<p>


## <center> <a name="intro_2"/> KDC Installation and Configuration
**Step 1:** To install packages for a Kerberos server <br>
```bash
# yum install -y krb5-server krb5-libs krb5-auth-dialog krb5-workstation
```

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
<code>kdb5_util create -r ***SEBC.SIN*** -s</code><p>

**Step 6:** Create Kerberos Administrator Principal<br>
<code>kadmin.local</code><br>
<code>kadmin.local: addprinc ***admin/admin@SEBC.SIN***</code><p>

**Step 7:** Start the Kerberos Daemons on the KDC<br>
<code>chkconfig krb5kdc on </code><br>
<code>chkconfig kadmin on </code><br>
<code>service krb5kdc start</code><br>
<code>service kadmin start</code><p>

**Step 8:** Test the Kerberos<br>
<code>kinit ***admin/admin@SEBC.SIN***</code><br>
<code>klist -e</code><p>

**Step 9:** Install the Kerberos client and utilities on all other cluster nodes<br>
<code>yum install -y krb5-libs krb5-auth-dialog krb5-workstation</code><p>

**Step 10:** Install the OpenLDAP client on Cloudera Manager node<br>
<code>yum install -y openldap-clients</code><p>



