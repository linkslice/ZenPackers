=====================================================================
Instructions for Compiling and starting Starting Serviced A-la-Carte
=====================================================================

Docker Prep
-------------
sudo docker login -u zenossinc+alphaeval -e "alpha2@zenoss.com" -p WP0FHD2M9VIKIX6NUXKTUQO23ZEWNSJLGDBA3SGEK4BLAI66HN5EU0BOKN4FVMFF https://quay.io/v1/
docker pull quay.io/zenossinc/opentsdb
docker pull quay.io/zenossinc/hbase
docker pull quay.io/zenossinc/daily-zenoss5-resmgr:5.0.0_389
# See: ImageID in http://artifacts.zenoss.loc/europa/NNN/zenoss5-resmgr-5.0.0_NNN.json for clues


Start Serviced: 
----------------

* zendev use europa
* cdz serviced
* sudo ./serviced/serviced -master -agent
* Add a Host::

   export IP_ADDRESS=$(ifconfig eth0 | grep 'inet addr:'| cut -d: -f2 | awk '{ print $1}')
   ./serviced/serviced host add $IP_ADDRESS:4979 default

Compile Template:
-------------------

* ./serviced/serviced template compile $(zendev root)/build/services/Zenoss.resmgr > /tmp/Zenoss.xxx.tpl
   serviced template compile --map zenoss/zenoss5x,quay.io/zenossinc/daily-whatever:5.0.0_389 \
   /path/to/Zenoss.resmgr

  ( or pipe it into the following command )
 
Add the Template:
------------------

* TEMPLATE_ID=$(serviced/serviced template add /tmp/Zenoss.xxx.tpl)
  (and Grab the TEMPLATE_ID number that results)
 

Deploy the Template:
--------------------

Method 1: CLI
~~~~~~~~~~~~~~~
* serviced/serviced template deploy $TEMPLATE_ID default zebra
* serviced/serviced template deploy $TEMPLATE_ID default zenoss
* Do not use: Example: serviced/serviced template deploy bd8ea68b-c0ae-703e-9a16-719cc6f1e1d7 default spearmint


Method 2: GUI
~~~~~~~~~~~~~~~

* Go to the UI at: [PUBLIC_IP]. Log in as zenoss/zenoss. 
* Deploy template. 
* Done.

