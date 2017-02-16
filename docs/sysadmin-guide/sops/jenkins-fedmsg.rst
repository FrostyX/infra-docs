.. title: Jenkins Fedmsg SOP
.. slug: infra-jenkins-fedmsg
.. date: 2016-05-11
.. taxonomy: Contributors/Infrastructure

==================
Jenkins Fedmsg SOP
==================

Send information about Jenkins builds to fedmsg.

Contact Information
===================

Owner
	Ricky Elrod, Fedora Infrastructure Team
Contact
	#fedora-apps

Reinstalling when it disappears
===============================

For an as-of-yet unknown reason, the plugin sometimes seems to disappear,
though it still shows as "installed" on Jenkins.

To re-install it, grab `fedmsg.hpi` from `/srv/web/infra/bigfiles/jenkins`.
Go to the Jenkins web interface and log in. Click `Manage Jenkins` ->
`Manage Plugins` -> `Advanced`. Upload the plugin and on the page that comes
up, check the box to have Jenkins restart when running jobs are finished.

Configuration Values
====================

These are written here in case the Jenkins configuration ever gets lost.
This is how to configure the jenkins-fedmsg-emit plugin.

Assume the plugin is already installed.

Go to "Configure Jenkins" -> "System Configuration"

Towards the bottom, look for "Fedmsg Emitter"

Values:

Signing: Checked
Fedmsg Endpoint: tcp://209.132.181.16:9941
Environment Shortname: prod
Certificate File: /etc/pki/fedmsg/jenkins-jenkins.fedorainfracloud.org.crt
Keystore File: /etc/pki/fedmsg/jenkins-jenkins.fedorainfracloud.org.key
