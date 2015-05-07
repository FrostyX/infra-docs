.. title: Fedora Status Service SOP
.. slug: infra-fedora-status
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

===========================
Fedora Status Service - SOP
===========================

Fedora-Status is the software that generates the page at
http://status.fedoraproject.org/. This page should be kept
up to date with the current status of the services ran by
Fedora Infrastructure.

This page is hosted at an OpenShift instance.
The upstream repository is fedora-status on FedoraHosted.org.

Contact Information
-------------------

Owner:
  Fedora Infrastructure Team
Contact:
  #fedora-admin, #fedora-noc
Servers:
  An OpenShift instance
Purpose: 
  Give status information to users about the current
  status of our public services.
Upstream:  
  http://git.fedorahosted.org/git/fedora-status.git

How it works
------------
To keep this website as stable as can be, the page is
generated at the time of upload by OpenShift.
 
As soon as you push to the OpenShift repo, a build hook
will create the HTML page.
 
Only members of sysadmin-noc and sysadmin-main can update
the status website.

Updating the page
-----------------
1. Check out the repo at::
      
    ssh://bab5ba6eb9b94f2083fdeefc5e87309b@status-fedora2.rhcloud.com/~/git/status.git/
 
2. cd status
3. Run ./manage.py

manage.py takes 3+ arguments::

[status] "[short summary message]" [service] ([service] .....)

[service] values can be found on http://status.fedoraproject.org/ on the RIGHT
SIDE of the header of each box. Examples are: 'wiki', 'pkgdb', and 'fedmsg'.

You can use "-" (dash) to imply "All services"

It accepts any number of additional services

[status] should be::

'major'     - A major service ourage.
'minor'     - A minor service outage (e.g. limited/geographical outage)
'scheduled' - The current downtime to this service is related to a scheduled outage
'good'      - Everything is fine and the service is functioning 100%.

[short summary message] is what appears in the body of the box and should tell
users what is happening/why the service is down.

You can use "-" (dash) to imply "Everything seems to be working." as the
status.

Examples::

./manage.py major "We're performing maintenance on the wiki database" wiki
./manage.py zodbot minor "Some IRC channels are having issues doing XYZ." zodbot
./manage.py good - -   # Set all services to good/default.
./manage.py good - wiki # Set wiki status to 'good' with the default message.
./manage.py good - wiki zodbot  # Set both wiki and zodbot to good with default message.

You can use the --general-info flag to set a "global" message, which appears
under the main status bar at the top of the page. Use this for big events that
effect all services, or to announce things like upcoming outages.
