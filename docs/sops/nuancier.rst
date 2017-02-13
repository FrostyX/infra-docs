.. title: Nuancier SOP
.. slug: infra-nuancier
.. date: 2016-03-11
.. taxonomy: Contributors/Infrastructure

=============
Nuancier SOP
=============

Nuancier is the web application used by the design team and the community to
submit and vote on the supplemental wallpapers provided with each version of
Fedora.

Contents
========

1. Contact Information
2. Documentation Links

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	https://apps.fedoraproject.org/nuancier
Servers
    nuancier01, nuancier02, nuancier01.stg, nuancier02.stg
Purpose
    Provide a system to submit and vote on supplemental wallpapers


Create a new election
=====================

* Login

* Go to the `Admin` panel via the menu at the top

* Click on `Create a new election`.

* Complete the form:

  Election name
    A short name used in all the pages, most often since we have one election
    per release it has been of the form `Fedora XX`

  Name of the folder containing the pictures
    This just links the election with the folder where the images will be
    uploaded on disk. Keep it simple, safe, something like `fXX` will do.

  Year
    The year when the election will be happening, this will just give some quick
    sorting option

  Submission start date (in UTC)
    The date from which the people will be able to submit wallpapers for the
    election. The submission starts on the exact day at midnight UTC.

  Start date (in UTC)
    The date when the election starts (and thus the submissions end). There is
    no buffer between when the submissions end and when the votes start which
    means admins have to keep up with the submissions as they are done.

  End date (in UTC)
    The date when the election ends. There are no embargo on the results, they
    are available right after the election ends.

  URL to claim a badge for voting
    The URL at which someone can claim a badge. This URL is displayed on the
    voting page as well as ones people have voted. Which means that having the
    badge does not ensure people voted, at max it ensures people visited
    nuancier during a voting phase.

  Number of votes an user can make
    The number of wallpapers an user can choose/vote on. This was made as they
    was a debate in the design team if having everyone vote on all 16 wallpapers
    was a good idea or not.

  Number of candidate an user can upload
    Restricts the number of wallpapers an user can submit for an election to
    prevent people from uploading tens of wallpapers in one election.

Review an election
==================

Admins must do that regularly during a submission phase to avoid candidates from
piling up.

* Login

* Go to the `Admin` panel via the menu at the top

* Find the election of interest in the list and click on `Review`

If the images are not showing, you can generate the thumbnails using the button
`(Re-)generate cache`.

On the review page, you will be able to filter the candidates by `Approved`,
`Pending`, `Rejected` or see them `All` (default).

You can then check the images one by one, select their checkbox and then either
`Approve` or `Deny` all the ones you selected.

.. note:: Rejections must be motivated in the `Reason for rejection / Comments`
          input field. This  motivation is then sent by email to the user
          explaining why a wallpaper they submitted was not accepted into the
          election.


Vote on an election
===================

Once an election is opened, a link announcing it will be available from the front
page and in the page listing the elections (`Elections` tab in the menu) a green
check-mark will appear on the `Votes` column while a red forbidden sign will
appear on the `Submissions` column.

You can then click on the election name which will take you on the voting page.

There, enlarge the image by clicking on them and make your choice by clicking on
the bottom right corner of the image.

On the column on the right the total number of vote available will appear.
If you need to change remove a wallpaper from your selection, simply click on it
in the right column.

As long as you have not picked the maximum number of candidates allowed, you can
cast your vote multiple times (but not on the same candidates of course).


View all the candidates of an election
======================================

All the candidates of an election are only accessible once the election is over.
If you wish to see all the images uploaded, simply go to the `Elections` tab and
click on the election name.


View the results of an election
===============================

The results of an election are accessible immediately after the end of it.
To see them, simply click the `Results` tab in the menu.

There you can click on the name of the election to see the wallpaper ordered by
their number of votes or on `stats` to view some stats about the election (such
as the number of participants, the number of voters, votes or the evolution of
the votes over time).


Miscellaneous
=============

Nuancier uses a gluster volume shared between the two hosts (in prod and in stg)
where are stored the images, making sure they are available to both frontends.
This may make things a little trickier sometime, be aware of it.
