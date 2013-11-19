========================
Dealing with Databases
========================

This chapter covers various Zenoss database problems and how to cure them.

Corrupted Mysql Partition File
-------------------------------

If you see this in your **/opt/zenoss/log/zeneventserver.log** file::

     SQL state [HY000]; error code [1696]; Failed to read from the .par file;
     nested exception is java.sql.SQLException: 
     Failed to read from the .par file at ..........

then there is a chance you have a corrupted Mysql database.

You may have to do the following to heal it:

* Stop Mysql
* Drop the zenoss_zep database (you may have to remove /var/lib/zenoss_zep)
* Start Mysql
* Recreate the zenoss_zep database::

  [zenoss@monitor:~]: zeneventserver-create-db --dbtype=mysql

* Restart Zenoss
* Test


