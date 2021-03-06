.. title: Voting and Elections Infrastructure SOP
.. slug: infra-voting
.. date: 2014-07-10
.. taxonomy: Contributors/Infrastructure

=========================
Voting Infrastructure SOP
=========================

The live voting instance can be found at
https://admin.fedoraproject.org/voting and the staging instance at
https://admin.stg.fedoraproject.org/voting/

The code base can be found at
http://git.fedorahosted.org/git/?p=elections.git

Contents
========

1. Contact Information
2. Creating a new election

  1. Creating the election
  2. Adding Candidates
  3. Who can vote

3. Modifying an Election

  1. Changing the details of an Election
  2. Removing a candidate
  3. Releasing the results of an embargoed election

Contact Information
===================

Owner
 Fedora Infrastructure Team
Contact
 #fedora-admin, elections
Location
 PHX
Servers
 elections0{1,2}, elections01.stg, db02
Purpose
 Provides a system for voting on Fedora matters

Creating a new election
=======================

Creating the elections
----------------------

* Log in

* Go to "Admin" in the menu at the top, select "Create new election" and fill
  in the form.

* The "usefas" option results in candidate names being looked up as FAS
  usernames an displayed as their real name.

* An alias should be added when creating a new election as this is used in
  the link on the page of listed elections on the frontpage.
  
* Complete the election form:

  Alias
    A short name for the election. It is the name that will be
    used in the templates.
    
    ``Example: FESCo2014``
   
  Summary
    A simple name that will be used in the URLs and as in the
    links in the application
 
    ``Example: FESCo elections 2014``
   
  Description
    A short description about the elections that will be
    displayed above the choices in the voting page
  
  Type
    Allow setting the types of elections (more on that below)
   
  Maxium Range/Votes  
    Allow setting options for some election type 
    (more on that below)
   
  URL
    A URL pointing to more information about the election
    
    ``Example: the wiki page presenting the election``

  Start Date
    The Start of the elections (UTC)
   
  End Date
    The Close of the elections (UTC)

  Number Elected
    The number of seats that will be selected among the candidates after the election

  Candidates are FAS users?  
    Checkbox allowing integration between FAS account
    and their names retrieved from FAS.
   
  Embargo results   
    If this is set then it will require manual intervention
    to release the results of the election
 
  Legal voters groups
    Used to restrict the votes to one or more FAS groups.

  Admin groups
    Give admin rights on that election to one or more FAS groups


Adding Candidates
=================

The list of all the elections can be found at voting/admin/

Click on the election of interest and and select "Add a candidate".

Each candidate is added with a name and an URL. The name can be his/her FAS username 
(interesting if the checkbox that candidates are FAS users has been checked when creating the calendar) or something else.

The URL can be a reference to the wiki page where they nominated themselves.

This will add extra candidates to the available list.

Who can vote
============

If no 'Legal voters groups' have been defined when creating the election, the
election will be opened to anyone having signed the CLA and being in one
other group (commonly referred to CLA+1).

Modifying an Election
=====================

Changing the details of an Election

.. note::
  this page can also be used to verify details of an election before it opens for voting.

The list of all the elections can be found at ``/voting/admin/``

After finding the right election, click on it to have the overview and select
"Edit election" under the description.

Edit a candidate
================

On the election overview page found via ``/voting/admin/`` (and clicking on the
election of interest), next to each candidate is an `[edit]` button allowing
the admins to edit the information relative to the candidate.

Removing a candidate
====================
   
On the election overview page found via ``/voting/admin/`` (and clicking on the
election of interest), next to each candidate is an `[x]` button allowing
the admins to remove the candidatei from the election.


Releasing the results of an embargoed election
==============================================

Visit the elections admin interface and edit the election to uncheck the
'Embargo results?' checkbox.

Results
=======

Admins have early access to the results of the elections (regardless of the
embargo status).

The list of the closed elections can be found at /voting/archives.

Find there the election of interest and click on the "Results" link in the
last column of the table.
This will show you the Results page included who was elected based on the
number of seats elected entered when creating the election.

You may use these information to send out the results email.

Legacy
======

.. note:: 
  The information below should now be included in the Results page (see above) 
  but I left them here in case.

Other things you might need to query
------------------------------------

The current election software doesn't retrieve all of the information that
we like to include in our results emails.  So we have to query the database
for the extra information.  You can use something like this to retrieve the
total number of voters for the election::

  ELECT e.id, e.shortdesc, COUNT(distinct v.voter) FROM elections AS e LEFT
  JOIN votes AS v ON e.id=v.election_id WHERE e.shortdesc in ('FAmSCo - February
  2014') GROUP BY e.id, e.shortdesc;


You may also want to include the vote tally per candidate for convenience
when the FPL emails the election results::

  SELECT e.id, e.shortdesc, c.name, c.novotes FROM elections AS e LEFT JOIN
  fvotecount AS c ON e.id=c.election_id WHERE e.shortdesc in ('FAmSCo - February
  2014', 'FESCo - February 2014') ;

