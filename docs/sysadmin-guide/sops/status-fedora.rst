.. title: Fedora Status Service SOP
.. slug: infra-fedora-status
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

===========================
Fedora Status Service - SOP
===========================

Fedora-Status is the software that generates the page at
http://status.fedoraproject.org/. This page should be kept
up to date with the current status of the services ran by
Fedora Infrastructure.

This page is hosted at AWS.
The upstream repository is fedora-status on Pagure.io.

Contact Information
===================

Owner:
  Fedora Infrastructure Team
Contact:
  #fedora-admin, #fedora-noc
Servers:
  AWS S3/CloudFront
Purpose: 
  Give status information to users about the current
  status of our public services.
Upstream:  
  https://pagure.io/fedora-status

How it works
============
To keep this website as stable as can be, the page is
hosted external to our main infrastructure, in AWS.

It is based on an S3 bucket with the files, fronted by
a CloudFront distribution for TLS termination and CNAMEs.

The files are statically generated on update, and then pushed
out to S3.

Only members of sysadmin-main and people given the AWS credentials
can update the status website.  Additionally, you also need to have
commit access to "statusfpo" git repository on GitHub - one way to get
it is becoming member of "noc" team at "fedora-infra" organization.

Setting up
==========
1. Check out the repo at::
      
    git@github.com:fedora-infra/statusfpo.git

2. Install locally the following packages: python2-jinja2 awscli

3. Grab ansible-private/files/aws-status-credentials and store in ~/.aws/credentials.

4. Run::

    aws configure set preview.cloudfront true

Updating the page
=================
 
1. cd statusfpo
2. Run ./manage.py

manage.py takes 3+ arguments::

[status] "[short summary message]" [service] ([service] .....)

[service] values can be found on http://status.fedoraproject.org/ on the RIGHT
SIDE of the header of each box. Examples are: 'wiki', 'pkgs', and 'fedmsg'.

You can use "-" (dash) to imply "All services"

It accepts any number of additional services

[status] should be::

'major'     - A major service outage.
'minor'     - A minor service outage (e.g. limited/geographical outage)
'scheduled' - The current downtime to this service is related to a scheduled outage
'good'      - Everything is fine and the service is functioning 100%.

[short summary message] is what appears in the body of the box and should tell
users what is happening/why the service is down.

You can use "-" (dash) to imply "Everything seems to be working." as the
status.

Examples::

./manage.py major "We're performing maintenance on the wiki database" wiki
./manage.py zodbot minor "Some IRC channels are having issues doing XYZ." zodbot
./manage.py good - -   # Set all services to good/default.
./manage.py good - wiki # Set wiki status to 'good' with the default message.
./manage.py good - wiki zodbot  # Set both wiki and zodbot to good with default message.

You can use the --general-info flag to set a "global" message, which appears
under the main status bar at the top of the page. Use this for big events that
effect all services, or to announce things like upcoming outages.

Renewing SSL certificate
========================

Because

1. Run certbot to generate certificate and have it signed by
   LetsEncrypt (you can run this command anywhere certbot is
   installed, you can use your laptop or
   certgetter01.phx2.fedoraproject.org)::

    rm -rf ~/certbot
    certbot certonly --agree-tos -m admin@fedoraproject.org --no-eff-email --manual --manual-public-ip-logging-ok -d status.fedoraproject.org -d www.fedorastatus.org --preferred-challenges http-01 --config-dir ~/certbot/conf --work-dir ~/certbot/work --logs-dir ~/certbot/log

2. You will be asked to make specific file available under specific
   URL.  In a different terminal upload requested file to AWS S3 bucket::

    echo SOME_VALUE >myfile
    aws --profile statusfpo s3 cp myfile s3://status.fedoraproject.org/.well-known/acme-challenge/SOME_FILE

3. Verify that uploaded file is available under the rigt URL.  If
   previous certificate already expired you may need to run curl with
   -k option::

    curl -kL http://www.fedorastatus.org/.well-known/acme-challenge/SOME_FILE

4. After making sure that curl outputs expected value, go back to
   certbot run and continue by pressing Enter.  You will be asked to
   repeat steps 2 and 3 for another domain.  Note that S3 bucket name
   should stay the same.

5. Deploy generated certificate to AWS.  This requires additional
   permissions on AWS.
