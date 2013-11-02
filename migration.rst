==============================================================================
Migration Guide for Slackers
==============================================================================

Description
------------------------------------------------------------------------------

Zenpacks that get upgraded often need some sort of migration of configuration 
data from the older version to the new. This is achieved via migration scripts
that are described in this doc.

Prerequisites
------------------------------------------------------------------------------

* Zenoss ZenPack Developement 
* Python 2.7

We assume that you are familiar with ZenPack developement and Python coding.
We further assume that we work from the base of ZP_DIR. 
For NetBotz for example:

export ZP_DIR_TOP=$ZENHOME/ZenPacks/ZenPacks.training.NetBotz
export ZP_DIR=$ZP_DIR_TOP/ZenPacks/training/NetBotz

Relative to this folder all migration scripts will reside in 

$ZP_DIR/mitrate/

Migration Outline
------------------------------------------------------------------------------
This migration guide is specific to a template-oriented ZenPack.
It may not be relevant to a different type of ZP.

The Basic idea behind this migration scenario is as follows:

* Identify the old zenpack objects, bound to a device.

   - Find all device classess and devices where "Oracle" template is bound.
   - Look for all **zDeviceTemplates** that are overridden

* Extract the old information from those templates

   - Leave them bound, for continuity sake.
   - Enable our modeler plugin for device class or device 
     (uses zCollectorPlugins)

* Populate the New ZP Data Structures

   - Create new Instance components from the old templates 
   - Populuate the new instances or components with data
   - As always, test your migration script by installing the new ZP over
     the old.

4. Give Users Instructions on Removing Old Object Templates

   - Since you may have left the old ZP objects in tact, 
     provide documentation on how to un-bind the old templates. 


Implementation
--------------

The most basic usage will involve creating a json input and then specifying a destination
prefix for the ZenPack. The first example is for an older (2.2) migration script.

In this example_.  we have a simple migration of the
zOraclePassword property to the new ZP. It only needs to set this single
zProperty value. In order to do so, it uses 
.. _Products.ZenModel.migrate.MigrateUtils.migratePropertyType() : https://github.com/zenoss/pm-resmgr-4.2.4/blob/master/src/core/Products/ZenModel/migrate/MigrateUtils.py


In our secont second-example_ we see a much more significant
migration() implementation. 

Specific Example: Zenpack.zenoss.DatabaseMonitor
-------------------------------------------------

This document will provide specific use case that will
hopefully guide you onward through the fog.

.. _example:

The Older 2.2 Migration
------------------------

OracleDB::

 In this example OracleDB is a device that inherits its /Device base from the
 parent server, be it Linux, AIX, Solaris, or some "other" operating system.
 This means that it needs to be able to patch itself underneath the device tree
 of that server target type and not have a stand-alone device root.

 The basic code strategy is to create a class that has a "migrate" method.
 The migrate() method gets called automatically by the ZenPack Installer.
 The first example is a very old version that does nothing but migrate 
 the password from one version to the next:

MigratePassword.py::

   ##############################################################################
   # Copyright (C) Zenoss, Inc. 2009, all rights reserved.
   ##############################################################################

   import logging
   log = logging.getLogger("zen.migrate")

   import Globals
   from Products.ZenModel.migrate.Migrate import Version
   from Products.ZenModel.ZenPack import ZenPackMigration
   from Products.ZenModel.migrate.MigrateUtils import migratePropertyType

   class MigratePassword(ZenPackMigration):
       version = Version(2, 2, 0)

       def migrate(self, dmd):
           log.info("Migrating zOraclePassword")
           migratePropertyType("zOraclePassword", dmd, "string")
           
   MigratePassword()

Notice that there is a "version" line just after the class definition. 
This version must identify the new version number of the ZP being migrated to.

The migration() method is very simple; in fact just one line that uses
the migratePropertyType() method to migrate the zOraclePassword.

.. _second-example:

The Newer 3.0 Migration
------------------------


This migration is more complex: It is no longer just template that binds to a
device, but it now is a component (Instance). You don't need about handling
component binding because that is taken care of by the actual modeler.

AddInstances.py::

   ############################################################################
   # Copyright (C) Zenoss, Inc. 2013, all rights reserved.
   ############################################################################

   import logging
   log = logging.getLogger("zen.migrate")

   from Products.ZenModel.DeviceClass import DeviceClass
   from Products.ZenModel.migrate.Migrate import Version
   from Products.ZenModel.ZenPack import ZenPackMigration

   # You must have the 'Oracle' template bound for migration to work
   TEMPLATE_NAME = 'Oracle'
   MODELER_PLUGIN_NAME = 'zenoss.ojdbc.Instances'

   def name_for_thing(thing):
      ''' Helper function to provide the name of the Device or DeviceClass '''

       if isinstance(thing, DeviceClass):
           return thing.getOrganizerName()

       return thing.titleOrId()

   class AddInstances(ZenPackMigration):
       '''
       Main class that contains the migrate() method. Note version setting.
       '''
       version = Version(3, 0, 0)

       def migrate(self, dmd):
           ''' 
           This is the main method. Its searches for overridden objects (templates)
           and then migrates the data to the new format or properties.
           In this case bound objects get assigned the new modeler pluging.
           '''
           overridden_on = dmd.Devices.getOverriddenObjects(
               'zDeviceTemplates', showDevices=True)

           for thing in overridden_on:
               if TEMPLATE_NAME in thing.zDeviceTemplates:
                   self.enable_plugin(thing)
                   self.populate_connection_strings(thing)

       def enable_plugin(self, thing):
           ''' Associate a collector plugin with the thing we have found.
               zCollectorPlugins is used by ModelerService.createDeviceProxy() 
               to add associated (modeler) plugins to the list for self-discovery.
               ModelerService.remote_getDeviceConfig() actually calls the modelers.
           '''
           current_plugins = thing.zCollectorPlugins
           if MODELER_PLUGIN_NAME in current_plugins:
               return

           log.info(
               "Adding %s modeler plugin to %s",
               MODELER_PLUGIN_NAME, name_for_thing(thing))

           current_plugins.append(MODELER_PLUGIN_NAME)
           thing.setZenProperty('zCollectorPlugins', current_plugins)

       def populate_connection_strings(self, thing):
           ''' Just a helper method to collect data for this ZP '''
           if thing.zOracleConnectionStrings:
               return

           connection_string = (
               'jdbc:oracle:thin:'
               '${here/zOracleUser}'
               '/${here/zOraclePassword}'
               '@${here/manageIp}'
               ':${here/zOraclePort}'
               ':${here/zOracleInstance}'
               )

           log.info(
               "Setting zOracleConnectionStrings for %s",
               name_for_thing(thing))

           thing.setZenProperty('zOracleConnectionStrings', [connection_string])

   AddInstances()
      
