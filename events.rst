=====================================
Events and Their Consequences
=====================================

Putting Events in Things
--------------------------

If you'd like to create an event in a modeler you can add these lines::

    import zope.component
    self._eventService = zope.component.queryUtility(IEventService)
    ........
    self._eventService.sendEvent(evt)

Here is an example ::

   def _deviceUp(self, result):
       msg = 'WMI connection to %s up.' % self._devId
       self._eventService.sendEvent(dict(
           summary=msg,
           eventClass=Status_Wmi,
           device=self._devId,
           severity=Clear,
           component='zenwin'))
       return result

   def _failure(self, result):
       """ Errback for an unsuccessful asynchronous connection. """

       err = result.getErrorMessage()
       log.error("Unable to scan device %s: %s", self._devId, err)

       self._reset()

       summary = """ Could not read Wins services (%s). Check your 6 """ % err

       self._eventService.sendEvent(dict(
           summary=summary,
           component='zenwin',
           eventClass=Status_Wmi,
           device=self._devId,
           severity=Error,
           ))

       return result


