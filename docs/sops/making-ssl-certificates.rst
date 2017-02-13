.. title: Infrastructure SSL Certificate Creation SOP
.. slug: infra-ssl-create
.. date: 2012-07-17
.. taxonomy: Contributors/Infrastructure

============================
SSL Certificate Creation SOP
============================

Every now and then you will need to create an SSL certificate for a
Fedora Service.

Creating a CSR for a new server.
================================

Know your hostname, ie `lists.fedoraproject.org``::

  export ssl_name=<fqdn of host> 


Create the cert. 8192 does not work with various boxes so we use 4096 currently.::
  
  openssl genrsa -out ${ssl_name}.pem 4096
  openssl req -new  -key ${ssl_name}.pem -out $(ssl_name}.csr

  Country Name (2 letter code) [XX]:US
  State or Province Name (full name) []:NM
  Locality Name (eg, city) [Default City]:Raleigh
  Organization Name (eg, company) [Default Company Ltd]:Red Hat
  Organizational Unit Name (eg, section) []:Fedora Project
  Common Name (eg, your name or your server's hostname)
  []:lists.fedorahosted.org
  Email Address []:admin@fedoraproject.org

  Please enter the following 'extra' attributes
  to be sent with your certificate request
  A challenge password []:
  An optional company name []:

send the CSR to the signing authority and wait for a cert.
place all three into private directory so that you can make certs in
the future.

Creating a temporary self-signed certificate.
=============================================

Repeat the steps above but add in the following::

  openssl x509 -req -days 30 -in ${ssl_name}.csr -signkey ${ssl_name}.pem -out ${ssl_name}.cert
   Signature ok
   subject=/C=US/ST=NM/L=Raleigh/O=Red Hat/OU=Fedora
   Project/CN=lists.fedorahosted.org/emailAddress=admin@fedoraproject.org

Getting Private key

We only want a self-signed certificate to be good for a short time so 30
days sounds good.
