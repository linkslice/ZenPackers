========================================================================
Collectors and Parsers
========================================================================

Collecting data is the fundamental goal of Zenoss. This section explores
specific tasks related to data collection and parsing that data.

Prerequisites
------------------------------------------------------------------------------

* Zenoss ZenPack Developement Guide

We assume that you are familiar with ZenPack developement and Python coding.
We further assume that we work from the base of ZP_DIR.
For NetBotz for example::

  ZP_DIR_TOP=$ZENHOME/ZenPacks/ZenPacks.training.NetBotz
  ZP_DIR=$ZP_DIR_TOP/ZenPacks/training/NetBotz

As you should know, collectors and parsers typically live in the folder::

  $ZP_DIR/datasources
  $ZP_DIR/parsers

Debugging Tips in General
---------------------------------------------------
* Run the collector manually like this::

   zencommand run  -workers=0 -v10 -d mp3.zenoss.loc |& tee collect.log

* If you don't get any output, you can try these basic steps:

  - Restart zenhub: it may have given up loading the modeler
  - Rerun zencommand and also monitor /opt/zenoss/log/zenhub.log for good
    measure. You may want to run zenhub in the foreground.

* To test your parser command at a low level use::

    cd /opt/zenoss/Products/ZenHub/services
    python CommandPerformanceConfig.py -d mp1.zenoss.loc

General Introduction
------------------------------------------------------------------------

Collection generally has two phases for collection,

*polling* phase and *collection* phase. In polling, the data is requested
and returned to the calling environment.


