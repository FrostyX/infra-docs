.. title: Fedora Infrastructure Github SOP
.. slug: infra-githup
.. date: 2014-09-26
.. taxonomy: Contributors/Infrastructure

===============================
Using github for Infra Projects
===============================

We're presently using github to host git repositories and issue tracking for
some infrastructure projects.  Anything we need to know should be recorded
here.

Setting up a new repo
=====================

Create projects inside of the fedora-infra group:

https://github.com/fedora-infra

That will allow us to more easily track what projects we have.

[TODO] How do we create a new project and import it?

- After creating a new repo, click on the Settings tab to set up some fancy
  things.

  If using git-flow for your project:
  
  - Set the default branch from 'master' to 'develop'.  Having the default
    branch be develop is nice:  new contributors will automatically start
    committing there if they're not paying attention to what branch they're
    on.  You almost never want to commit directly to the master branch.

    If there does not exist a develop branch, you should create one by
    branching off of master.::

        $ git clone GIT_URL
        $ git checkout -b develop
        $ git push --all

  - Set up an IRC hook for notifications.  From the "settings" tab click on
    "Webhooks & Services."  Under the "Add Service" dropdown, find "IRC" and
    click it. You might need to enter your password.
    In the form, you probably want the following values:

    - Server, irc.freenode.net
    - Port, 6697
    - Room, #fedora-apps
    - Nick, <nothing>
    - Branch Regexes, <nothing>
    - Password, <nothing>
    - Ssl, <on>
    - Message Without Join, <on>
    - No Colors, <off>
    - Long Url, <off>
    - Notice, <on>
    - Active, <on>


Add an EasyFix label
====================

The EasyFix label is used to mark bugs that are potentially fixable by new
contributors getting used to our source code or relatively new to python
programming.  GitHub doesn't provide this label automatically so we have to
add it.  You can add the label from the issues page of the repository or use
this curl command to add it::

  curl -k -u '$GITHUB_USERNAME:$GITHUB_PASSWORD' https://api.github.com/repos/fedora-infra/python-fedora/labels -H "Content-Type: application/json" -d '{"name":"EasyFix","color":"3b6eb4"}'

Please try to use the same color for consistency between Fedora Infrastructure
Projects.  You can then add the github repo to the list that
easyfix.fedoraproject.org scans for easyfix tickets here:

https://fedoraproject.org/wiki/Easyfix
