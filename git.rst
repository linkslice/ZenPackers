========================================================================
Git for Gits ;)
========================================================================

Git is a developement version control tool. 

Beginning
------------------------------------------------------------------------
To just pull (download) a repository from the web:

* Find the repo online and get the clone string.
* Copy that string.
* pull it down with "git clone"

::
  
  <bash>: git clone git@github.com:zenoss/ZenPackers.git

    - or for https - 

  <bash>: git clone https://github.com/zenoss/ZenPackers.git

Typical Workflow Scenario
--------------------------------------------------------------

Now that you have a repo, go into the repo folder.

* Add any files you want
* Make any changes you want to files
* Commit your changes
* Push your changes

To add your new files::

  <bash>: git add -A

To commit all changes execute::

  <bash>: git commit -a

To finally push up your changes to your Repo hub (or github.com)::

  <bash>: git push

New Repo Workflow Scenario
--------------------------------------------------------------

* First go to github.com and create your account
* Then create an empty repository in the GUI
* Now on your workstation, pull down (clone) the empty repo::

  <bash>: git clone https://github.com/zenoss/bogus.git
  <bash>: cd bogus/

* Now start writing your code, make files......
* Now add files to your repo and push::

   <bash>: git add -A
   <bash>: git commit -a
   <bash>: git push
   <bash>: git status
   .. Already up-to-date ..

Setting Repo Parameters
----------------------------------------------

* Configure the Username and Email for the Repo::

  <bash>: git config --global user.name "Joe Frazer"
  <bash>: git config --global user.email joe@zenoss.com

* Reset the Author for the Repo::

  <bash>: git commit --amend --reset-author

Changing Branches
-------------------------

* Change branch from **master** to **develop** with *checkout*::

  <bash>: git checkout develop
  <bash>: git status

Merging Branches
-------------------------

You like the work you've done in develop and think it should be merged into master.
You can do this by using the *merge* option.

* First change branches from develop to master::

  <bash>: git checkout master

Now things are as before with master in its original state. 

* Now you want to merge from develop::

   <bash>: git merge develop
     Updating 1530600..2873dc4
     Fast-forward
     .gitignore                             |    2 +
     Makefile                               |   11 +-
     ....

* Now you must push these changes up to your Hub::

   <bash>: git push
     Total 0 (delta 0), reused 0 (delta 0)
     To git@github.com:zenoss/ZenPackers.git
     1530600..2873dc4  master -> master


Delete Unwanted Branches
------------------------
If you want to eject unwanted branches from your repo,
make sure to read the git-branch docs and the warnings about being
fully merged (--delete option).

To remove a local branch::

  git branch -D <branchName>

To remove a  remote branch::
  
  git push origin --delete <branchName>


Comparison of Git Branches
---------------------------------------------------

* Show only relevant commits between two git refs::

   git log --no-merges master..develop

=============================================================================
Git Flow 
=============================================================================

Git flow simplifies development revisioning.
http://danielkummer.github.io/git-flow-cheatsheet/

Setup Git Flow in the Existing Repo
------------------------------------
::

   <bash>: git flow init

Create New Features and Work Flow
----------------------------------
In features, you don't want to use version numbers because it can
cause chaos when multiple authors work the same project. Instead
give the version a name, and only after the resulting develop is 
reviewed, you give it a version. (Source Unknown: Rob B).

To start a new feature::

  <bash>: git flow feature start xyz
  <bash>: git status
   On branch feature/xyz (don't give version #'s)
   nothing to commit (working directory clean)
   
   .... do some work ....
   .... do some more work ....
   .... you are finished ....

  <bash>: git commit -a 
  <bash>: git push (nothing happens)
  - (At this point you can ask for a Pull Request or continue)
  <bash>: git flow feature finish xyz
  <bash>: git status
   On branch develop
   nothing to commit (working directory clean)

Now you are back on develop. You still need to push your changes up::

  <bash>: git push
   Total 0 (delta 0), reused 0 (delta 0)
   To git@github.com:zenoss/ZenPackers.git
   1530600..2873dc4  develop -> develop


Feature Drop from Develop to Feature/XYZ
-----------------------------------------

So you have a fix in develop that needs to be pulled into your feature/xyz branch.
You will merge **develop** into feature/xyz

* From your feature branch feature/xyz, make sure you commit and push::

  <bash>: git commit -a 
  <bash>: git push

* Now merge from develop::

  <bash>: git merge develop
  <bash>: git push origin develop
 
* You may have to deal with merge conflicts as this point.


Push the Develop onto the old Feature that is Stale
----------------------------------------------------
You have created a branch (forgotten) that has been left behind and wish upgrade
it with all the new changes that have been made with other feature enhancements.
You don't have anything to save in it. Use these commands (with caution)
to merge develop back onto feature/forgotten::

  <bash>: git checkout feature/forgotten
  <bash>: git push . develop:feature/forgotten
  <bash>: get checkout feature/forgotten
  <bash>: git commit -a
  <bash>: git push

Push a new Feature up to Origin for storage:
-----------------------------------------------------
Sometimes you want a feature to be stored on your Hub.
Git Flow does not automatically push your features.
You can push it up to the hub like this::

  <bash>: git push -u origin feature/new

Git Stash: Stashing Modified Files
------------------------------------

Git's *stash* option allows you to put modified files into a temporary holding
area. The usual scenario is to stash your mods away then pull from the origin,
and then re-place your stash'ed files into the tree. Then you can push the 
results back up to origin. Here is a possible workflow::

  .... you made changes to develop, but you'd rather it be in a feature....

  <bash>: git stash
   > Saved working directory and index state WIP on develop: e38b798 post
   release: 1.0.1 -> 1.0.2dev.....

  <bash>: git flow feature start cleanup_on_aisle_7
   > Switched to a new branch 'feature/cleanup_on_aisle_7'

  <bash>: git stash pop
  .... now you have your new mods overlaid ....
  .... make whatever other modifications ....
  .... now you can commit all your mods ....

  <bash>: git commit -a

  <bash>: git flow feature finish cleanup_on_aisle_7

  <bash>: git push

Pull Requests: The Easy Way
----------------------------

The easiest way we have to get your code reviewed and merged into a major
branch is to use Git Flow to create a feature, push that feature up to Github,
and have someone review it. 

Here is the workflow in a nutshell:

* Create your feature with **git flow**
* Make your mods
* Commit your mods
* Push your feature up to Gitflow
* Go into the Github GUI, select your feature
* Make your pull request
* Ask for a review
* That reveiwer then **merges** your changes into develop
* Finsh your feature locally: Everything on Github is cleaned up for you.



