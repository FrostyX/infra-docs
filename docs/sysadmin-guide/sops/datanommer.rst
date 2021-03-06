.. title: Datanommer SOP 
.. slug: infra-datanommer
.. date: 2013-02-08
.. taxonomy: Contributors/Infrastructure

==============
datanommer SOP
==============

Consume fedmsg bus activity and stuff it in a postgresql db.

Contact Information
===================

Owner
  Messaging SIG, Fedora Infrastructure Team
Contact
  #fedora-apps, #fedora-fedmsg, #fedora-admin, #fedora-noc
Servers
  busgateway01
Purpose
  Save fedmsg bus activity

Description
===========

datanommer is a set of three modules:

python-datanommer-models 
  Schema definition and API for storing new items
  and querying existing items
  
python-datanommer-consumer 
  A plugin for the fedmsg-hub that actively
  listens to the bus and stores events.
  
datanommer-commands
  A set of CLI tools for querying the DB.

datanommer will one day serve as a backend for future web services like
datagrepper and dataviewer. 

Source: https://github.com/fedora-infra/datanommer/
Plan: https://fedoraproject.org/wiki/User:Ianweller/statistics_plus_plus

CLI tools
=========

Dump the db into a file as json::

    $ datanommer-dump > datanommer-dump.json

When was the last bodhi message?::

    $ # It was 678 seconds ago
    $ datanommer-latest --category bodhi --timesince
    [678]

When was the last bodhi message in more readable terms?::

    $ # It was 12 minutes and 43 seconds ago
    $ datanommer-latest --category bodhi --timesince --human
    [0:12:43.087949]

What was that last bodhi message?::

    $ datanommer-latest --category bodhi
    [{"bodhi": {
      "topic": "org.fedoraproject.stg.bodhi.update.comment", 
      "msg": {
        "comment": {
          "group": null, 
          "author": "ralph", 
          "text": "Testing for latest datanommer.", 
          "karma": 0, 
          "anonymous": false, 
          "timestamp": 1360349639.0, 
          "update_title": "xmonad-0.10-10.fc17"
        }, 
        "agent": "ralph"
      }, 
    }}]

Show me stats on datanommer messages by topic::

    $ datanommer-stats --topic
    org.fedoraproject.stg.fas.group.member.remove has 10 entries
    org.fedoraproject.stg.logger.log has 76 entries
    org.fedoraproject.stg.bodhi.update.comment has 5 entries
    org.fedoraproject.stg.busmon.colorized-messages has 10 entries
    org.fedoraproject.stg.fas.user.update has 10 entries
    org.fedoraproject.stg.wiki.article.edit has 106 entries
    org.fedoraproject.stg.fas.user.create has 3 entries
    org.fedoraproject.stg.bodhitest.testing has 4 entries
    org.fedoraproject.stg.fedoratagger.tag.create has 9 entries
    org.fedoraproject.stg.fedoratagger.user.rank.update has 5 entries
    org.fedoraproject.stg.wiki.upload.complete has 1 entries
    org.fedoraproject.stg.fas.group.member.sponsor has 6 entries
    org.fedoraproject.stg.fedoratagger.tag.update has 1 entries
    org.fedoraproject.stg.fas.group.member.apply has 17 entries
    org.fedoraproject.stg.__main__.testing has 1 entries

Upgrading the DB Schema
=======================

datanommer uses "python-alembic" to manage its schema.  When developers want
to add new columns or features, these should/must be tracked in alembic and
shipped with the RPM.

In order to run upgrades on our stg/prod dbs:

1) ssh to busgateway01{.stg}
2) ``cd /usr/share/datanommer.models/``
3) Run::

    $ alembic upgrade +1

  Over and over again until the db is fully upgraded.
