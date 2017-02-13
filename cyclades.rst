.. title: cyclades
.. slug: infra-cyclades
.. date: 2011-12-12
.. taxonomy: Contributors/Infrastructure
========
Cyclades
========

cyclades notes

1. login as root - default password is tslinux
2. change password for root and admin to our password from the
     phx2-access.txt file in the private repo
3. port forward to the web browser for the cyclades
    ``ssh -L 8080:rack47-serial.phx2.fedoraproject.org:80``
4. connect to localhost:8080 in your web browser
5. login with root and the password you set above
6. click on 'security'
7. click on 'moderate'
8. logout, port forward port 443 as above:
    ``ssh -L 8080:rack47-serial.phx2.fedoraproject.org:443``
9. click on the 'wizard' button at lower left
10. proceed through the wizard
    Info needed:
    - serial ports are set to 115200 8N1 by default
    - do not setup buffering
    - give it the ip of our syslog server

11. click 'apply changes'
12. hope
13. log back in
14. name/setup the port aliases

