=====================================
Zenpack Release Notes
=====================================

We have a formal process for releasing Zenpacks.
The steps are:

* 3rd Party Software approval
* Ensure an appropriate copyright is included in each source file
* Set the Version
* Release to Master
* Publish the Documentation on Wiki


3rd Party Software Approval
--------------------------------

This is covered above. Please see that document.

Appropriate copyright for each Source File
--------------------------------------------

Something similar to this should be in each sourcecode file::

   ############################################################################
   #
   # Copyright (C) Zenoss, Inc. 2014 all rights reserved.
   #
   # This content is made available according to terms specified in
   # License.zenoss under the directory where your Zenoss product is installed.
   #
   ############################################################################

Set the Version
----------------

First pick a version: We recommend using a format x.y.z, typically starting at
1.0.0 (though you can start at 0.x.y if you want.) The first digit is the major
version, the second is the minor version, and the third is the revision.

Set this version in the $ZP_HOME/setup.py file. We use these version number
variables as examples::

   * OLD=1.0.0       (The previous Master release)
   * CURRENT=1.0.1   (The previous Master release)
   * NEW=1.0.2       (The new Develop branch)

Tag the Release
----------------

Release to Master
------------------
We use *Git Flow* to release with some custom conventions:

First we select a RELEASE name according to version name.


Checkout Master and Develop
-----------------------------

Starting from the *develop* branch

* git flow release start $CURRENT
* Edit setup.py (set the correct version numbers)
* Commit:

  - For new release: git commit -a -m "release: version $CURRENT"
  - For update release: git commit -a -m "release: version $OLD -> $CURRENT"

* git flow release finish $CURRENT

  - You will be prompted for the *Commit String*: "tag $CURRENT"
  - You will automatically pushed back into develop

* update develop setup.py

  - with bump and dev: Example: $CURRENT -> ${NEW}dev

* Commit again: 

  - git commit -a -m "post release: $OLD -> ${CURRENT}dev"

* git push
* git push ---tags

  - This tags the revision with a crpyto-secure key for reference

* Finally, push up the master changes:
  
  - git checkout master
  - git push
  - git checkout develop


Build the Master on Jenkins
---------------------------

Got to master branch on Jenkins and build it.
This will look like

http://jenkins.zenosslabs.com/job/master-ZenPacks.zenoss.XYZ/

Parature
--------------
Note: Mere mortals are not typically allowed to do this step.
      Consult Chet, John, or Rusty.

Method A: Required
~~~~~~~~~~~~~~~~~~~
* Email the ZP to Rusty: rwilson@zenoss.com

Method B: Deprecated (Do this if you are suicidal).
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Go into Parature
* Download the master Jenkins build from above: master-ZenPacks.zenoss.XYZ
* Upload the egg to Parature: Parature->downloads->bulkUpload
* Move the egg to the latest RM Zenpacks directory ( like 4.2.4 RM Zenpacks etc)
* Settings:

  - Perm: Platinum
  - Perm: Gold
  - Perm: Silver
  - Perm: Bronze
  - published

* Copy download link (or number)
* Paste that link to the approproate place in the Wiki


