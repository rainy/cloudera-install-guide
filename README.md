# Cloudera Install Guide
This guide explains how to install Cloudera Manager, CDH and MIT Kerberos. Cloudera Manager 5 supports managing CDH 4 and CDH 5.


## Cloudera Manager Requirements
* **Supported Operating Systems**
  * RHEL-compatible systems
    * Red Hat Enterprise Linux and CentOS 5.7, 64-bit
    * Red Hat Enterprise Linux and CentOS 6.4, 64-bit
    * Red Hat Enterprise Linux and CentOS 6.4 in SE Linux Mode
    * Red Hat Enterprise Linux and CentOS 6.5, 64-bit
    * Oracle Enterprise Linux 5.6 (UEK R2), 64-bit
    * Oracle Enterprise Linux 6.4 (UEK R2), 64-bit
    * Oracle Enterprise Linux 6.5 (UEK R2, UEK R3), 64-bit

* **Supported JDK Versions**
  * Cloudera Manager supports Oracle JDK 1.7.0_55 when it's managing CDH 5.x

* **Supported Browsers**
  * Firefox 24 or 31
  * Google Chrome
  * Internet Explorer 9 or later
  * Safari 5 or later

* **Supported Databases**
  * MySQL - 5.0, 5.1, 5.5, and 5.6
  * Oracle 11gR2
  * PostgreSQL - 8.4, 9.1, and 9.2

* **Resource Requirements**
  * **Disk Space**
    * **Cloudera Manager Server**
      * 5 GB on the partition hosting /var.
      * 500 MB on the partition hosting /usr.
      * In the local parcel repository on the Cloudera Manager Server the approximate sizes of the various parcels are as follows:
        * CDH 4.6 - ~700 MB per parcel, CDH 5 - ~1 GB per parcel
        * Impala - ~200 MB per parcel
        * Solr - ~ 400 MB per parcel
    * **Cloudera Management Service** - The Host Monitor and Service Monitor databases are stored on the partition hosting /var. Ensure that you have at least 20 GB available on this partition. For further information, see Data Storage for Monitoring Data.
    * **Agents** - On Agent hosts each unpacked parcel requires about three times the space of the downloaded parcel on the Cloudera Manager Server. By default unpacked parcels are located in /opt/cloudera/parcels.
  * **RAM** - 4 GB is appropriate for most cases, and is required when using Oracle databases. 
  * **Python** - Cloudera Manager uses Python. All supported operating systems contain a Python version 2.4 or higher. Cloudera Manager and CDH 4 require at least Python 2.4, but Hue in CDH 5 requires Python 2.6 or 2.7.


## **Follow the steps [documented here](https://github.com/rainy/cloudera-install-guide/blob/master/Cloudera%20Installation.md)**
