.. title: WhatCanIDoForFedora.org SOP
.. slug: wcidff.org
.. date: 2017-04-17
.. taxonomy: Contributors/Infrastructure

============================
What Can I Do For Fedora SOP
============================

Contents
========

1. Contact Information
2. Introduction
3. Determine Category
4. Cron Job

Contact Information
===================

Owner
	sysadmin-main
Contact
	#fedora-admin, #fedora-noc or admin@fedoraproject.org
Location
	RedHat Openshift via Amazon AWS @ 54.172.181.217
Server(s)
	apps.fedoraproject.org
Purpose
    To explain the overall function of the whatCanIDoForFedora webpage.. including some back story,
    how to build your own, and site navigation.


Introduction
============

The 'What Can You Do For Fedora' (whatcanyoudoforfedora.org) page was the brainchild of Ralph Bean after
getting inspiration from 'whatcanidoformozilla.org' created by Josh Matthews, Henri Koivuneva and a few
others.  Ralph wanted to make the whatcanidoforfedora (wcidff) as configurable as possible.  The purpose
of this site is to assist, in as user friendly a way as possible, new and prospective community members
and help them realize what skills they may posess that can be helpful for the Fedora Project.

Requirements
============

The site-generator is written in Python, so you will need that installed and working. You will also need
mako and PyYAML.

You can cover all of these via::

  $ sudo yum install python-mako PyYAML python-virtualenv

The script can optionally generate an 'svg' visualizing your question tree. This requires the pygraphviz
package which can be installed like::

  $ sudo yum install python-pygraphviz

Beginning
=========

Once you've met the requirements above, Clone the repo::

  $ git clone https://github.com/fedora-infra/asknot-ng.git
  $ cd asknot-ng

Create a virtualenv into which you can install the module::

  $ virtualenv --system-site-packages venv
  $ source venv/bin/activate
  $ python setup.py develop

Run the script with the Fedora configuration::

  $ ./asknot-ng.py templates/index.html questions/fedora.yml l10n/fedora/locale --theme fedora

Wrote build/en/index.html and open up build/en/index.html in your favorite browser.

Translations
============

First, setup a virtualenv, install Babel, and build the egg info::

 $ virtualenv venv
 $ source venv/bin/activate
 $ pip install Babel
 $ python setup.py develop

Then, extract the translatable strings::

  $ python setup.py extract_messages --output-file l10n/fedora/locale/asknot-ng.pot

Deploying to OpenShift
======================

This is a very easy way to bring asknot-ng to a production server using OpenShift. (Provided you have an OS account)

Just create a new application using RHC and set the environment variables to your demands::

  rhc app create askorg diy --from-code https://github.com/fedora-infra/asknot-ng#develop
  rhc set-env ASKNOT_THEME=org -a askorg
  rhc set-env ASKNOT_QUESTION_FILE=questions/org.yml -a askorg
  rhc app restart askorg

Further Details
===============

Wcidff has many facets, offers a multitude of interest areas for the public to contribute.  Each 'area' links the user to a
'sub-area' where the user can provide a more granular choice.  See below for a detailed explanation::

    Design----------------Design Team
           ---------------Websites Team

    Coding----------------Python
           ---------------C
           ---------------Haskell
           ---------------Java
           ---------------Scala
           ---------------JavaScript
           ---------------Ruby
           ---------------C++

    CommOps---------------General
            --------------Specific

    Writing---------------Documentation
            --------------Blogging

    Categorization--------Fedora Tagger

    Translation-----------General
                ----------Specific

    Advocacy--------------Ambassador

    Packaging-------------Package Maintainer

    QA Testing------------General
               -----------Specific

    Modularization--------Error:Depreciated Page:
    https://fedoraproject.org/wiki/Modularity/Getting_Started/Get_involved?rd=Modularity/Getting_involved

    The Desktop-----------Desktop Sig
                ----------Cinnamon Sig
                ----------KDE Sig
                ----------LXDE Sig
                ----------XFCE Sig

    The Server------------Server Sig

    The Cloud-------------Cloud Sig
              ------------RDO

    Internationalization--I18N
