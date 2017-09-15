.. title: AWS Access SOP
.. slug: infra-aws-access
.. date: 2017-07-28
.. taxonomy: Contributors/Infrastructure

==========================
Amazon Web Services Access
==========================

AWS includes a highly granular set of access policies, which can be
combined into roles and groups.  Ipsilon is used to translate between
IAM policy groupings and groups in the Fedora Account System (FAS).


Contact Information
===================

Owner
    Fedora Infrastructure Team
Contact
    #fedora-admin
Persons
    puiterwijk, nirik, pfrields
Location
    ?
Servers
    N/A
Purpose
    Provide AWS resource access to contributors via FAS group membership.

    
Accessing the AWS Console
=========================

To access the AWS Console via Ipsilon authentication, use `this SAML
link
<https://id.fedoraproject.org/saml2/SSO/Redirect?SPIdentifier=urn:amazon:webservices&RelayState=https://console.aws.amazon.com>`_.

You must be in the `aws-iam FAS group
<https://admin.fedoraproject.org/accounts/group/view/aws-iam>`_ (or
another group with IAM access) to perform this action.


Adding a role to AWS IAM
------------------------

Sign into AWS via the URL above, and visit `Identity and Access
Management (IAM) <https://console.aws.amazon.com/iam/home>`_ in the
Security, Identity and Compliance tools.

Choose Roles to view current roles.  Confirm there is not already a
role matching the one you need.  If not, create a new role as follows:

1. Select *Create role*.
2. Select *SAML 2.0 federation*.
3. Choose the SAML provider *id.fedoraproject.org*, which should
   already be populated as a choice from previous use.
4. Select the attribute *SAML:aud*. For value, enter
   *https://signin.aws.amazon.com/saml*. Do not add a
   condition. Proceed to the next step.
5. Assign the appropriate policies from the pre-existing IAM policies.
   It's unlikely you'll have to create your own, which is outside the
   scope of this SOP.  Then proceed to the next step.
6. Set the role name and description.  It is recommended you use the
   *same* role name as the FAS group for clarity.  Fill in a longer
   description to clarify the purpose of the role. Then choose *Create
   role*.

Note or copy the Role ARN (Amazon Resource Name) for the new role.
You'll need this in the mapping below.


Adding a group to FAS
---------------------

When finished, login to FAS and create a group to correspond to the
new role.  Use the prefix *aws-* to denote new AWS roles in FAS.  This
makes them easier to locate in a search.

It may be appropriate to set group ownership for *aws-* groups to an
Infrastructure team principal, and then add others as users or
sponsors.  This is especially worth considering for groups that have
modify (full) access to an AWS resource.


Adding an IAM role mapping in Ipsilon
-------------------------------------

Add the new role mapping for FAS group to Role ARN in the ansible git
repo, under *roles/ipsilon/files/infofas.py*.  Current mappings look
like this::

  aws_groups = {
      'aws-master': 'arn:aws:iam::125523088429:role/aws-master',
      'aws-iam': 'arn:aws:iam::125523088429:role/aws-iam',
      'aws-billing': 'arn:aws:iam::125523088429:role/aws-billing',
      'aws-atomic': 'arn:aws:iam::125523088429:role/aws-atomic',
      'aws-s3-readonly': 'arn:aws:iam::125523088429:role/aws-s3-readonly'
  }

Add your mapping to the dictionary as shown.  Ipsilon should receive
the change within a few minutes of the ansible git repo change.


Other Notes
===========

AWS resource access that is not read-only should be treated with care.
In some cases, Amazon or other entities may absorb AWS costs, so
changes in usage can cause issues if not controlled or monitored.  If
you have doubts about access, consult the Fedora Project Leader or
Fedora Engineering Manager.
