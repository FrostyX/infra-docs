.. title: Ask Fedora SOP 
.. slug: infra-ask-fedora
.. date: 2017-05-15
.. taxonomy: Contributors/Infrastructure

==============
Ask Fedora SOP
==============

To set up https://ask.fedoraproject.org based on Askbot as a question and
answer support forum for the Fedora community. A prodcution instance could be
seen at https://ask.fedoraproject.org and the staging instance is at
http://ask.stg.fedoraproject.org/

This page describes how to set up and customize it from scratch.

Contents                                 
========

1. Contact Information                
2. Creating database                  
3. Setting up the forum               
4. Adding administrators              
5. Change settings within the forum   
6. Database tweaks
7. Debugging                          


Contact Information
===================

Owner
	Fedora Infrastructure Team
Contact
	#fedora-admin
Persons
	anyone from the sysadmin team
Sponsor
	nirik
Location
	phx2
Servers
	ask01 , ask01.stg
Purpose
	To host Ask Fedora


Creating database
=================

We use the postgresql database backend. To add the database to a
postgresql server::

   # psql -U postgres
   postgres# create user askfedora with password 'xxx';
   postgres# create database askfedora;
   postgres# ALTER DATABASE askfedora owner to askfedora;
   postgres# \q;

Now setup the db tables if this is a new install::

   python manage.py syncdb
   python manage.py migrate askbot
   python manage.py migrate django_authopenid #embedded login application


Setting up the forum
====================

Askbot is packaged and available in Rawhide, Fedora 16 and EPEL 6. On a
RHEL 6 system, you need to install EPEL 6 repo first.::

   # yum install askbot

The /etc/askbot/sites/ask/conf/settings.py file should look something
like::

   DATABASE_ENGINE = 'postgresql_psycopg2'
   DATABASE_NAME = 'testaskbot'
   DATABASE_USER = 'askbot'
   DATABASE_PASSWORD = 'xxxxx'
   DATABASE_HOST = '127.0.0.1'
   DATABASE_PORT = '5432'

   # Outgoing mail server settings
   #
   DEFAULT_FROM_EMAIL = 'askfedora@fedoraproject.org'
   EMAIL_SUBJECT_PREFIX = '[Askfedora]'
   EMAIL_HOST='127.0.0.1'
   EMAIL_PORT='25'

   # This variable points to the Askbot plugin which will be used for user
   # authentication.  Not enabled yet because we don't need FAS auth but use 
   # Fedora id as a openid provider.
   #
   # ASKBOT_CUSTOM_AUTH_MODULE = 'authfas'

   Now Ask Fedora website should be accessible from the browser.


Adding administrators
=====================

As of Askbot version 0.7.21, the first user who logs in automatically
becomes the administrator. In previous versions, you have to do the
following.::

  # cd /etc/askbot/sites/ask/conf/
  # python manage.py add_admin 1
     Do you really wish to make user (id=1, name=pjp) a site administrator?
     yes/no: yes

Once a user is marked as a administrator, he or she can go into anyone's
profile, go the "moderation" tab in the end and mark them as administrator
or moderator as well as block or suspend a user.


Change settings within the forum
================================

* Data entry and display:
	- Disable "Allow asking questions anonymously"
	- Enable "Force lowercase the tags"
	- Change "Format of tag list" to "cloud"
	- Change "Minimum length of search term for Ajax search" to "3"
	- Change "Number of questions to list by default" to "50"
	- Change "What should "unanswered question" mean?" to "Question has no
	- answers"
	 
* Email and email alert settings
	- Change "Default news notification frequency" to "Instantly"

* Flatpages - about, privacy policy, etc.
   Change "Text of the Q&A forum About page (html format)" to the following::

      Ask Fedora provides a community edited knowledge base and support forum
      for the Fedora community. Make sure you read the FAQ and search for
      existing questions before asking yours. If you want to provide feedback,
      just a question in this site! Tag your questions "meta" to highlight your
      questions to the administrators of Ask Fedora.

* Login provider settings
   - Disable "Activate local login"

* Q&A forum website parameters and urls
   - Change "Site title for the Q&A forum" to "Ask Fedora: Community Knowledge
      Base and Support Forum"
   - Change "Comma separated list of Q&A site keywords" to "Ask Fedora, forum,
      community, support, help"
   - Change "Copyright message to show in the footer" to "All content is under
      Creative Commons Attribution Share Alike License. Ask Fedora is community
      maintained and Red Hat or Fedora Project is not responsible for content"
   - Change "Site description for the search engines" to "Ask Fedora: Community
      Knowledge Base and Support Forum"
   - Change "Short name for your Q&A forum" to "Ask Fedora"
   - Change "Base URL for your Q&A forum, must start with http or https" to
      "http://ask.fedoraproject.org"

* Sidebar widget settings - main page
   - Disable "Show avatar block in sidebar"
   - Disable "Show tag selector in sidebar"

* Skin and User Interface settings
  - Upload "Q&A site logo"
  - Upload "Site favicon". Must be a ICO format file because that is the only one IE supports as a fav icon.
  - Enable "Apply custom style sheet (CSS)"
  - Upload the following custom CSS::

      #ab-main-nav a {
      color: #333333;
      background-color: #d8dfeb;
      border: 1px solid #888888;
      border-bottom: none;
      padding: 0px 12px 3px 12px;
      height: 25px;
      line-height: 30px;
      margin-right: 10px;
      font-size: 18px;
      font-weight: 100;
      text-decoration: none;
      display: block;
      float: left;
      }

      #ab-main-nav a.on {
      height: 24px;
      line-height: 28px;
      border-bottom: 1px solid #0a57a4;
      border-right: 1px solid #0a57a4;
      border-top: 1px solid #0a57a4;
      border-left: 1px solid #0a57a4; /*background:#A31E39; */
      background: #0a57a4;
      color: #FFF;
      font-weight: 800;
      text-decoration: none
      }

      #ab-main-nav a.special {
      font-size: 18px;
      color: #072b61;
      font-weight: bold;
      text-decoration: none;
      }

      /* tabs stuff */
      .tabsA { float: right; }
      .tabsC { float: left; }
       
      .tabsA a.on, .tabsC a.on, .tabsA a:hover, .tabsC a:hover {
      background: #fff;
      color: #072b61;
      border-top: 1px solid #babdb6;
      border-left: 1px solid #babdb6;
      border-right: 1px solid #888a85;
      border-bottom: 1px solid #888a85;
      height: 24px;
      line-height: 26px;
      margin-top: 3px;
      }
       
      .tabsA a.rev.on, tabsA a.rev.on:hover {
      padding: 0px 2px 0px 7px;
      }
       
      .tabsA a, .tabsC a{
      background: #f9f7eb;
      border-top: 1px solid #eeeeec;
      border-left: 1px solid #eeeeec;
      border-right: 1px solid #a9aca5;
      border-bottom: 1px solid #888a85;
      color: #888a85;
      display: block;
      float: left;
      height: 20px;
      line-height: 22px;
      margin: 5px 0 0 4px;
      padding: 0 7px;
      text-decoration: none;
      }
       
      .tabsA .label, .tabsC .label {
      float: left;
      font-weight: bold;
      color: #777;
      margin: 8px 0 0 0px;
      }
       
      .tabsB a {
      background: #eee;
      border: 1px solid #eee;
      color: #777;
      display: block;
      float: left;
      height: 22px;
      line-height: 28px;
      margin: 5px 0px 0 4px;
      padding: 0 11px 0 11px;
      text-decoration: none;
      }

      a {
      color: #072b61;
      text-decoration: none;
      cursor: pointer;
      }

      div.side-box
      {
      width:200px;
      padding:10px;
      border:3px solid #CCCCCC;
      margin:0px;
      background: -moz-linear-gradient(top, #DDDDDD, #FFFFFF);
      }

Database tweaks
===============

To automatically delete expired sessions, we run a trigger
that makes PostgreSQL delete them upon inserting a new one.

The code used to create this trigger was::

  askfedora=# CREATE FUNCTION delete_old_sessions() RETURNS trigger
  askfedora-# LANGUAGE plpgsql
  askfedora-# AS $$
  askfedora$# BEGIN
  askfedora$# DELETE FROM django_session WHERE expire_date<current_timestamp;
  askfedora$# RETURN NEW;
  askfedora$# END
  askfedora$# $$;
  CREATE FUNCTION
  askfedora=# CREATE TRIGGER old_sessions_gc
  askfedora-# AFTER INSERT ON django_session
  askfedora-# EXECUTE PROCEDURE delete_old_sessions();

In case this trigger causes any problems, please remove it
by running: ``DROP TRIGGER old_sessions_gc;``
   
To make this perform, we have a custom index that's not in
upstream askbot, please remember to add that when recreating
the trigger::

  CREATE INDEX CONCURRENTLY django_session_expire_date ON django_session (expire_date);

If you deleted the trigger, or reinstalled without trigger,
please make sure to run ``manage.py clean_sessions`` regularly,
so you don't end up with a database that's too massive in
size.


Debugging
=========

Set DEBUG to True in settings.py file and restart Apache.


Auth issues
===========

Users can login to ask with a variety of social media accounts.
Once they login with one they can attach other ones as well. 

If a user forgets what social media they used, you can look in the
database: 

Login to database host (db01.phx2.fedoraproject.org)
# sudo -u postgres psql askfedora
psql> select * from django_authopenid_userassociation where user_id like '%username%';

If they can login again with the same auth, ask them to do so. 
If not, you can add the fedora account system openid auth to allow
them to login with that: 

psql> insert into django_authopenid_userassociation (user_id, openid_url,provider_name) VALUES
(2595, 'http://name.id.fedoraproject.org', 'fedoraproject');

Use the ID from the previous query and replace name with the users fas name
