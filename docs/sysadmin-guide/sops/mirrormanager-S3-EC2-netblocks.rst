.. title: Infrastructure AWS Mirroring SOP
.. slug: infra-aws-mirror
.. date: 2014-12-05
.. taxonomy: Contributors/Infrastructure

===========
AWS Mirrors
===========

Fedora Infrastructure mirrors EPEL content (/pub/epel) into Amazon
Simple Storage Service (S3) in multiple regions, to make it fast for
EC2 CentOS/RHEL users to get EPEL content from an effectively local
mirror.

For this to work, we have private mirror entries in MirrorManager, one
for each region, which include the EC2 netblocks for that region.

Amazon updates their list of network blocks roughly monthly, as they
consume additional address space.  Therefore, we need to make the
corresponding changes into MirrorManager's entries for same.

Amazon publishes their list of network blocks on their forum site,
with the subject "Announcement: Amazon EC2 Public IP Ranges".	As of
November 2014, this was
https://forums.aws.amazon.com/ann.jspa?annID=1701

As of November 19, 2014, Amazon publishes it as a JSON file we can download.
http://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html
