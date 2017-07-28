.. title: AWS Access SOP
.. slug: aws-access
.. date: 2017-07-28
.. taxonomy: Contributors/Infrastructure

==========================
Amazon Web Services Access
==========================

AWS includes a highly granular set of access policies, which can be
combined into roles and groups.  Ipsilon is used to translate between
IAM policy groupings and groups in the Fedora Account System (FAS).

To access the AWS Console via Ipsilon authentication, use this URL:
FIXME

------------------------
Adding a role to AWS IAM
------------------------

Sign into AWS via the URL above, and visit Identity and Access
Management (IAM) in the Security, Identity and Compliance tools.

https://console.aws.amazon.com/iam/home

FIXME

When finished, login to FAS and create a group to correspond to the
new role.  Use the prefix *aws-* to denote new AWS roles in FAS.  This
makes them easier to locate in a search.

-------------------------------------
Adding an IAM role mapping in Ipsilon
-------------------------------------

FIXME

