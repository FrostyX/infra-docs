.. title: Hotfixes SOP
.. slug: infra-hotfix
.. date: 2015-02-24
.. taxonomy: Contributors/Infrastructure

============
HOTFIXES SOP
============

From time to time we have to quickly patch a problem or issue
in applications in our infrastructure. This process allows
us to do that and track what changed and be ready to remove
it when the issue is fixed upstream.


Ansible based items:
====================
For ansible, they should be placed after the task that installs
the package to be changed or modified. Either in roles or tasks.

hotfix tasks should be called "HOTFIX description"
They should also link in comments to any upstream bug or ticket.
They should also have tags of 'hotfix'

The process is:

- Create a diff of any files changed in the fix.
- Check in the _original_ files and change to role/task
- Check in now your diffs of those same files.
- ansible will replace the files on the affected machines
  completely with the fixed versions.
- If you need to back it out, you can revert the diff step,
  wait and then remove the first checkin

Example::

  <task that installs the httpd package>

  #
  # install hash randomization hotfix
  # See bug https://bugzilla.redhat.com/show_bug.cgi?id=812398
  #
  - name: hotfix - copy over new httpd init script
    copy: src="{{ files }}/hotfix/httpd/httpd.init" dest=/etc/init.d/httpd
          owner=root group=root mode=0755
    notify:
    - restart apache
    tags:
    - config
    - hotfix
    - apache

Upstream changes
================

Also, if at all possible a bug should be filed with the upstream
application to get the fix in the next version. Hotfixes are something
we should strive to only carry a short time.
