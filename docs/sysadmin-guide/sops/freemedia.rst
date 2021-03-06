.. title: FreeMedia Infrastructure SOP
.. slug: infra-freemedia
.. date: 2014-12-18
.. taxonomy: Contributors/Infrastructure

============================
FreeMedia Infrastructure SOP
============================

This page is for defining the SOP for Fedora FreeMedia Program. This will
cover the infrastructural things as well as procedural things.

Contents
========

1. Location of Resources
2. Location on Ansible
3. Opening of the form
4. Closing of the Form
5. Tentative timeline
6. How to

  1. Open
  2. Close

7. Handling of tickets

  1. Login
  2. Rejecting Invalid Tickets
  3. Accepting Valid Tickets

8. Handling of non fulfilled requests
9. How to handle membership applications

Location of Resources
=====================
* The web form is at
  https://fedoraproject.org/freemedia/FreeMedia-form.html
* The TRAC is at [63]https://fedorahosted.org/freemedia/report

Location on ansible
===================

$PWD = ``roles/freemedia/files``

Freemedia form
	 FreeMedia-form.html
Backup form
	 FreeMedia-form.html.orig
Closed form
	 FreeMedia-close.html
Backend processing script
	 process.php
Error Document
	 FreeMedia-error.html

Opening of the form
===================

The form will be opened on the First day of each month.

Closing of the Form
===================

Tentative timeline
------------------

The form will be closed after a couple of days. This may vary according to
the capacity.

How to
======

* The form is available at
  ``roles/freemedia/files/FreeMedia-form.html`` and
  ``roles/freemedia/files//FreeMedia-form.html.orig``

* The closed form is at
  ``roles/freemedia/files/FreeMedia-close.html``

Open
----

* Goto roles/freemedia/tasks
* Open ``main.yml``
* Goto line 32.
* To Open: Change the line to read::
    src="FreeMedia-form.html"
* After opening the form, go to trac and grant "Ticket Create and
  Ticket View" privilege to "Anonymous".

Close
-----

* Goto roles/freemedia/tasks
* Open main.yml
* Goto line 32.
* To Close: Change the line to read::
    src="FreeMedia-close.html",
* After closing the form, go to trac and remove "Ticket Create and
    Ticket View" privilege from "Anonymous".

..  note::
  * Have to check about monthly cron.
  * Have to write about changing init.pp for closing and opening

Handling of tickets
===================

Login
-----

* Contributors are requested to visit
    https://fedorahosted.org/freemedia/report
* Please login with your FAS account.

Rejecting Invalid Tickets
-------------------------

* If a ticket is invalid, don't accept the request. Go to "resolve as:"
  and select "invalid" and then press "Submit Changes".

* A ticket is Invalid if

    * No Valid email-id is provided.
    * The region does not match the country.
    * No Proper Address is given.

* If a ticket is duplicate, accept one copy, close the others as
  duplicate Go to "resolve as:" and select "duplicate" and then press
  "Submit Changes".

Accepting Valid Tickets
-----------------------
* If you wish to fulfill a request, please ensure it from the above
  section, it is not liable to be discarded.

* Now "Accept" the ticket from the "Action" field at the bottom, and
  press the "Submit Changes" button.

* These accepted tickets will be available from
  https://fedorahosted.org/freemedia/report user both "My Tickets"
  and "Accepted Tickets for XX" (XX= your region e.g APAC)

* When You ship the request, please go to the ticket again, go to
  "resolve as:" from the "Action" field and select "Fixed" and then
  press "Submit Changes".

* If an accepted ticket is not finalised by the end of the month, is
  should be closed with "shipping status unknown" in a comment

Handling of non fulfilled requests
----------------------------------

We shall close all the pending requests by the end of the Month.

* Please Check your region

How to handle membership applications
-------------------------------------

Steps to become member of Free-media Group.

1. Create an account in Fedora Account System (FAS)
2. Create an user page in Fedora Wiki with contact data. Like
    User:<nick-name>. There are templates.
3. Apply to Free-Media Group in FAS
4. Apply to Free-Media mailing list subscription

Rules for deciding over membership applications
````````````````````````````````````````````````
======= ================ ========== =============== =========================
Case    Applied to       User Page  Applied to             Action
        Free-Media Group Created    Free-Media List
======= ================ ========== =============== =========================
1       Yes               Yes       Yes             Approve Group and mailing
                                                    list applications
------- ---------------- ---------- --------------- -------------------------
                                                    Put on hold + Write to
2       Yes               Yes       No              subscribe to list Within
                                                    a Week
------- ---------------- ---------- --------------- -------------------------
                                                    Put on hold + Write to
3       Yes               No        whatever        make User Page Within a
                                                    Week
------- ---------------- ---------- --------------- -------------------------
4       No                No        Yes             Reject
======= ================ ========== =============== =========================

.. note::
    1. As you need to have an FAS account for steps 2 and 3, this is not
       included in the decision rules above
    2. The time to be on hold is one week. If not action is taken after one
       week, the application has to be rejected.
    3. When writing asking to fulfil steps, send CC to other Free-media
       sponsors to let them know the application has been reviewed.
