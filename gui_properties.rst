==============================================================================
GUI Properties and JavaScript Related Methods
==============================================================================

Description
------------------------------------------------------------------------------

Zenpack GUIs need to cute.
This is done by getting more properties in the GUI.
We explain how to do this here a bit...

Prerequisites
------------------------------------------------------------------------------

* Zenoss ZenPack Developement 
* Python 2.7

We assume that you are familiar with ZenPack developement and Python coding.
We further assume that you have a console/terminal/xterm open on the system
running Zenoss. The CLI default prompt is **[zenoss:~]:**

# Polling (TBD)
# Device Modeling (TBD)
# RRD (TBD)
# Event Traps (TBD)


Trick: Change Column Appearances
---------------------------------

Change the "First Seen" and "Last Seen" columns in the event console to show
how long ago the event occurred in a more human-friendly way. This is done 
through the following JS function exampe (time_ago_columns.js)::


   (function() {

       var time_ago_column = {
           renderer: function(value, metaData, record, rowIndex, colIndex, store) {
               var seconds = Math.floor((new Date() - value) / 1000);
               var interval = Math.floor(seconds / 31536000);

               if (interval > 1)
                   return interval + " years ago";

               interval = Math.floor(seconds / 2592000);
               if (interval > 1)
                   return interval + " months ago";

               interval = Math.floor(seconds / 86400);
               if (interval > 1)
                   return interval + " days ago";

               interval = Math.floor(seconds / 3600);
               if (interval > 1)
                   return interval + " hours ago";

               interval = Math.floor(seconds / 60);
               if (interval > 1)
                   return interval + " minutes ago";

               return Math.floor(seconds) + " seconds";
           }
       };

       Zenoss.events.registerCustomColumn('firstTime', time_ago_column);
       Zenoss.events.registerCustomColumn('lastTime', time_ago_column);

   }());
