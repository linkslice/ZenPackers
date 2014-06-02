Updating Serviced in Go
==========================

Execute these commands::

   zendev use europa
   export GOPATH=$(zendev root)/src/golang
   export PATH=$(zendev root)/src/golang/bin:${PATH}
   export ZENHOME=$(zendev root)/zenhome

Clear out old data::

  # stop serviced
  serviced.init stop
  sudo rm -rf /tmp/serviced-root

Normal Update Serviced::

  cdz serviced
  git pull
  make

Enhanced Update Serviced::

  # go into the europa environment
  cdz  serviced
  git status
  # ( This next line may be required if you can't pull properly )
  git checkout HEAD isvcs/resources/logstash/logstash.conf
  git pull
  make

Update Europa Build Environment (Before updating template )
-------------------------------------------------------------

Contains servive templates....

* cdz
* cd build
* git pull
* Now rebuild the template and re-deploy

Update Templates Method I: Map the template to match Docker
-------------------------------------------------------------

Build the Template::

  cdz serviced

  export ZVER='daily-zenoss5-resmgr:5.0.0_389'

  serviced template compile -map zenoss/zenoss5x,quay.io/zenossinc/$ZVER \
  $(zendev root)/build/services/Zenoss.resmgr > /tmp/x.tpl

  # ./serviced/serviced template compile $(zendev root)/build/services/Zenoss.resmgr > /tmp/x.tpl

  ( you probably need to start serviced now )
  TEMPLATE_ID=$(serviced/serviced template add /tmp/x.tpl)
  ./serviced/serviced host add $IP_ADDRESS:4979 default

Deploy Templates::

   serviced/serviced template deploy $TEMPLATE_ID default zenoss

Update Templates Method II: Tag Docker image to match template
---------------------------------------------------------------

This method pulls the docker image and tags it::

   # Alternative to mapping the template: Tag the image: 
   docker pull quay.io/zenossinc/daily-zenoss5-resmgr:5.0.0_389
   docker tag quay.io/zenossinc/daily-zenoss5-resmgr:5.0.0_389 zenoss/zenoss5x

   ./serviced/serviced host add $IP_ADDRESS:4979 default

You don't need to deploy the template since it already matches your docker image

Start Serviced  
----------------
You can do it one of 2 ways, I prefer the first, which 
requires youinstall serviced.init in your ~/bin. It also
logs to /tmp/serviced.log

* serviced.init start
* cdz serviced ; serviced/serviced -master -agent

