.. title: Fedora Planet Subgroup SOP
.. slug: infra-planet-subgroup
.. date: 2011-10-03
.. taxonomy: Contributors/Infrastructure

==================================
Planet Subgroup Infrastructure SOP
==================================

Fedora's planet infrastructure produces planet configs out of users'
``~/.planet`` files in their homedirs on fedorapeople.org. You can also create
subgroups of users into other planets. This document explains how to setup
new subgroups.

Contact Information
===================

Owner
  Fedora Infrastructure Team
Contact
  #fedora-admin
Servers
  batcave01/ planet.fedoraproject.org
Purpose
  provide easy setup of new planet groups on
  planet.fedoraproject.org

following:

The Setup

1. on batcave01::
   
    cp -a configs/system/planet/grouptmpl configs/system/planet/newgroupname

2. cd to the new directory

3. run::
   
    perl -pi -e "s/%%groupname/newgroupname/g" fpbuilder.conf base_config planet-group.cron templates/* 
   
  replacing newgroupname with the groupname you want

4. git add the whole dir

5. edit ``manifests/services/planet.pp``

6. copy and paste everything from begging to end of the design team group, to use as a template.

7. modify what you copied replacing design with the new group name

8. save it

9. check everything in

10. run ansible on planet and check if it works

Use
===

Tell the requester to then copy their current .planet file to
.planet.newgroupname. For example with the design team::

  cp ~/.planet ~/.planet.design

This will then show up on the new feed -
http://planet.fedoraproject.org/design/

