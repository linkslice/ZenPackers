========================================================================
Modeling for Zenpacks: Selected Topics
========================================================================

Modeling is an integral part of Zenoss. This article explores specific
tasks related to modeling.


Prerequisites
------------------------------------------------------------------------------

* Zenoss ZenPack Developement Guide

We assume that you are familiar with ZenPack developement and Python coding.
We further assume that we work from the base of ZP_DIR.
For NetBotz for example::

  ZP_DIR_TOP=$ZENHOME/ZenPacks/ZenPacks.training.NetBotz
  ZP_DIR=$ZP_DIR_TOP/ZenPacks/training/NetBotz

As you should know, modelers typically live in the folder::

  $ZP_DIR/modeler/plugins/zenoss/MyModeler.py

General Introduction
------------------------------------------------------------------------

Modeler classes generally have two methods that are used by the $zenmodeler$
service. They are:

* collect(): This method collects the data in an asychronous way.
  It returns a dict called results
  Its signature is typically::

      @inlineCallbacks
      def collect(self, device, log)
          ....
          returnValue(results)

* process(self, device, results, log):
  This method (asynchronously) takes that results dict uses it to populate
  the device model.

We talk more in the sections below about this.

Creating Containing Relation Maps from Modeled Data
---------------------------------------------------

Creation of the Relation Maps require several pieces:

* Base class instances must have containing relationships defined
* Modeler must correctly insert the components into the relationships defined.


Base Class Relations Definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In our example we'll use two classes: Instance and TableSpace from the Oracle
ZP. Instance is a component off of Device.Device, and Tablespace will hang
off of Instance. We need two defining relation:

* In Instance() we need two relations. The first
  binds Instance to Device.Device and the second give us
  multiple TableSpace to Instance::

    # Generic relations (from ZP Generator)
    _relations = ()
     for Klass in Klasses:
         _relations = _relations + getattr(Klass, '_relations', ())

    # These are the ones we need to define:
    _relations = _relations + (
         ('Instance_host',
              ToOne(ToManyCont,
                    'Products.ZenModel.Device.Device',
                    'oracle_instances',)),
         ('oracle_tablespaces',
              ToManyCont(
                  ToOne,
                  'ZenPacks.zenoss.DatabaseMonitor.TablesSpace.TableSpace',
                  'instance',)),
         )

* In TableSpace() we need just one to define Instance -> TablesSpaces::

    # Generic relations (from ZP Generator)
    _relations = ()
    for Klass in Klasses:
        _relations = _relations + getattr(Klass, '_relations', ())

    # This is the one we define.
    _relations = _relations + (
        ('instance', ToOne(ToManyCont,
                           'Products.ZenModel.Instance.Instance',
                           'oracle_tablespaces',
                             ),
        ),)

Modeler Class Relations Insertion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
We now discuss what is in your modeler (in our example, Instance) class.

In the collect() method in your modeler, we assume you have collected all the
required data and stored in in the results dictionary. How you do that is
fairly general.

In our Instance modeler's process method, we will first create a temporary
storage dictionary called *datamap*, which has Instance as a key and a list
of TableSpace objects as the values. Once this datamap is created and populated,
we then iterate through it to setup the RelationshipMap() structures.

To set this up we first loop through the results data to create the temporary
datamap::

   for connectionString, data in results.items():

       instance1 = data['instance'][0]
       instance_name = instance1.get('INSTANCE_NAME')
       ts_list = data['tablespaces']

       om = self.objectMap()
       om.id = self.prepId('orainst-%s' % instance_name
       om.title = instance_name

       tablespaces = []
       for ts in ts_list:
           tablespaces.append(ObjectMap(data=dict(
               id='{0}_{1}'.format(instance_name, ts['TABLESPACE_NAME'])
               tablespace_name = ts['TABLESPACE_NAME'],
               tablespace_instance = instance1.get('INSTANCE_ROLE'),
               tablespace_maxbytes = ts['BYTES_MAX'],
            )))


       # Add to map: Map the om object to the ts
       datamap[om] = tablespaces

So now you have your datamap setup. Its only used to feed our RelationshipMap.
Notice that in this example we must:

#. Get the list of Instances outside the loop using the dict.keys() for the
   Instance -> Device.Device relation.
#. We need to then loop over the Instances to attache the assiciated TableSpace
   list objects

::

       #------------------------------------------------------------------
       # Now loop over objects to create relation maps.
       #------------------------------------------------------------------

        relmaps = []

        relmaps.append(RelationshipMap(
            relname='oracle_instances',
            modname='ZenPacks.zenoss.DatabaseMonitor.Instance',
            objmaps=datamap.keys()))

        for inst, ts in datamap:
            print type(inst), type(ts)

            relmaps.append(RelationshipMap(
                compname='oracle_instances/{0}'.format(inst.id),
                relname='oracle_tablespaces',
                modname='ZenPacks.zenoss.DatabaseMonitor.TableSpace',
                objmaps=ts))


        log.info('%s: %s instances found', device.id, len(relmaps))
        return relmaps


This is a simple example. To see this how this was implemented see the
ZenPacks.zenoss.DatabaseMonitor's modeler plugin.

To see other examples: 

* ZenPacks.zenoss.PostgreSQL (simpler)
* ZenPacks.zenoss.XenServer  (more complex)

