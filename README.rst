====
WHAT
====

A fresh look on buildout, a playground. Managing releases more sanely, 
permissions, etc. This will be for a basic deploy to start, then work 
at easy extension for additoinal eggs.


===
WHY
===

Buildout as is traditionally used in Plone is shit for deploy tools. 
Things I'm hoping to fix:

 * Unix like directory: rename parts to etc, move things around to handle things 
   mentioned blow.
 * Permissions: With everything based out of one dir, permissions are wrong at best
   and overly permission at worst. the bin dir should r/w/e with user and group. The 
   egg dirs should only be readable by running process. the data dir needs stricter perms
   although I can't understand why blobcache is never running with the right perm 
   ANYWAYS. pids can hide, data should be obvious. Note that this was born from an 
   issue with puppet enforcing permissions. Because in traditional buildouts the eggs
   directory is under the buildout config dir it causes the permissions enforcement
   to spin like a mofo, often not completing. Permissions need to be clear and 
   enforcable.
 * Version control: The configs should be in version control. Nothing else.
 * Release management: deploying should be as easy as downloading the config dir 
   and watching it go. Buildouts should be versioned like eggs. Download the config
   and run the release. Reverting a release means switching to the old dir and you 
   are done. No need to wait for buildout
 * Log management: All should go to the syslog. Then what happens from there is up
   to you. 
 * Webhooks: Notify when buildout starts, runs, errs
 * Development packages: stop that shit for prod. Good egging process or at minimum
   not overriding that conflicting package on the system. A big bone in my butt on 
   automated release are dirty packages, accidentally or otherwise.
 * Best practice: if this is set up correctly, there should be a 1 paragraph 
   explanation on how to hook into puppet and github releases.
 * Extension: Add new eggs and sources by modifying 1 file, with the capacity to do
   this programatically. custom.cfg? Maybe... could be hosted remotely, and edited
   on the users side. INSERT SOMETHING CLEVER
 * On the same note, put together a collection of egg packages that fit different 
   domains. For example, edu.cfg.
 * Data: build in data backup to the deploy process. Repozo, build, restart always.
   Reverts won't be perfect, but they will be close enough. 
 * Production vs Dev: be clear on what is needed for production. Buildout should 
   never pick anything in prod but always fail in dev. Checkouts should always 
   happen, and at this moment I say mr.developer should never ever be used in 
   a production deploy.
 * Easy to clean up: no more guessing which files are deletable and which ones 
   aren't. If its not in deploy, they are all on the hitlist
 * supervisor: stays in its own little box. It messes things UP!


====
WHEN
====

It's a personal project. Don't hold your breath.


===
HOW
===

Production
^^^^^^^^^^

  1. Clone this repo
  2. Optional: make a virtual env in the root::

      % virtualenv python

 
  3. Go into deploy and set up everything. This only has to be done once.
     This will set up the directory structure, which will be symlinky later::

     % cd deploy
     % ../python/bin/python bootstrap.py


  4. Prepare the release::

     % ../bin/buildout -c release.cfg


  5. Prepare zope

     % ../bin/buildout -c prod.cfg



Logging
^^^^^^^

You'll note that in production there is no syslog set up. This is because any logging
is meant to be configured through rsyslog. This will make sure everything is centralized,
and that if you have other syslog setups they can work too.

To make this work properly, configure puppet to add a file /etc/rsyslog.d/30-zope.conf
with a link to where you want the logs, and any remote logging like papertrail::

    local3.*          @logs.papertrailapp.com:XXXXX
    local3.*          /var/log/zope.log


====
TODO
====

 * the only way to get buildout outside of the bin dir is to do ::

    > ../python/bin/python2.7 bootstrap.py -c buildout.cfg 

   So I think that there needs to be a prelim step which puts buildout in 
   the deploy directory, although maybe this can be tied into a better tool
