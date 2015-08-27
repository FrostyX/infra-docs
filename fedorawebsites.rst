.. title: Websites Release SOP
.. slug: infra-websites
.. date: 2015-08-27
.. taxonomy: Contributors/Infrastructure

===================
Webites Release SOP
===================


  * 1. Preparing the website for a release

    * 1.1 Obsolete GPG key of the EOL Fedora release
    * 1.2 Update GPG key
      * 1.2.1 Steps

  * 2. Update website

    * 2.1 For Alpha
    * 2.2 For Beta
    * 2.3 For GA

  * 3. Fire in the hole

  * 4. Tips

    * 4.1 Merging branches



  1. Preparing the website for a new release cycle

    1.1 Obsolete GPG key

    One month after a Fedora release the release number 'FXX-2' (i.e. 1 month
    after F21 release, F19 will be EOL) will be EOL (End of Life).
    At this point we should drop the GPG key from the list in verify/ and move
    the keys to the obsolete keys page in keys/obsolete.html.

    1.2 Update GPG key

    After another couple of weeks and as the next release approaches, watch
    the fedora-release package for a new key to be added. Use the update-gpg-keys
    script in the fedora-web git repository to add it to static/. Manually add it
    to /keys and /verify in all websites where we use these keys:
      * arm.fpo
      * getfedora.org
      * labs.fpo
      * spins.fpo

      1.2.1 Steps

      a) Get a copy of the new key(s) from the fedora-release repo, you will
         find FXX-primary and FXX-secondary keys. Save them in ./tools to make the
         update easier.

         https://pagure.io/fedora-repos

      b) Start by editing ./tools/update-gpg-keys and adding the key-ids of
         any obsolete keys to the obsolete_keys list.

      c) Then run that script to add the new key(s) to the fedora.gpg block:

         fedora-web git:(master) cd tools/
         tools git:(master) ./update-gpg-keys RPM-GPG-KEY-fedora-23-primary
         tools git:(master) ./update-gpg-keys RPM-GPG-KEY-fedora-23-secondary

         This will add the key(s) to the keyblock in static/fedora.gpg and
         create a text file for the key in static/$KEYID.txt as well. Verify
         that these files have been created properly and contain all the keys
         that they should.

         * Handy checks: gpg static/fedora.gpg or gpg static/$KEYID.txt
         * Adding "--with-fingerprint" option will add the fingerprint to the
           output

         The output of fedora.gpg should contain only the actual keys, not the
         obsolete keys.
         The single text files should contain the correct information for the
         uploaded key.

      d) Next, add new key(s) to the list in data/verify.html and move the new
         key informations in the keys page in data/content/keys/index.html. A
         script to aid in generating the HTML code for new keys is in
         ./tools/make-gpg-key-html.
         It will print HTML to stdout for each RPM-GPG-KEY-* file given as
         arguments. This is suitable for copy/paste (or directly importing if
         your editor supports this).
         Check the copied HTML code and select if the key info is for a primary
         or secondary key (output says 'Primary or Secondary').

         tools git:(master) ./make-gpg-key-html RPM-GPG-KEY-fedora-23-primary

         Build the website with 'make en test' and carefully verify that the
         data is correct. Please double check all keys in http://localhost:5000/en/keys
         and http://localhost:5000/en/verify.

         NOTE: the tool will give you an outdated output, adapt it to the new
         websites and bootstrap layout!


  2. Update website

    2.1 For Alpha

      a) Create the fXX-alpha branch from master
         fedora-web git:(master) git push origin master:refs/heads/f22-alpha

         and checkout to the new branch:
         fedora-web git:(master) git checkout -t -b f13-alpha origin/f13-alpha

      b) Update the global variables
         Change curr_state to Alpha for all arches

      c) Add Alpha banner
         Upload the FXX-Alpha banner to static/images/banners/f22alpha.png
         which should appear in every ${PRODUCT}/download/index.html page.
         Make sure the banner is shown in all sidebars, also in labs, spins, and arm.

      d) Check all Download links and paths in ${PRODUCT}/prerelease/index.html
         You can find all paths in bapp01 (sudo su - mirrormanager first) or
         you can look at the downlaod page http://dl.fedoraproject.org/pub/alt/stage

      e) Add CHECKSUM files to static/checksums and verify that the paths are
         correct. The files should be in sundries01 and you can query them with:
         $ find /pub/fedora/linux/releases/test/17-Alpha/ -type f -name \
         *CHECKSUM* -exec cp '{}' . \;
         Remember to add the right checksums to the right websites (same path).

      f) Add EC2 AMI IDs for Alpha. All IDs now are in the globalvar.py file.
         We get all data from there, even the redirect path to trac the AMI IDs.
         We now also have a script which is useful to get all the AMI IDs uploaded
         with fedimg. Execute it to get the latest uploads, but don't run the script too
         early, as new builds are added constantly.
         fedora-web git:(fXX-alpha) python ~/fedora-web/tools/get_ami.py

      g) Add CHECKSUM files also to http://spins.fedoraproject.org in
         static/checksums. Verify the paths are correct in data/content/verify.html.
         (see point e) to query them on sundries01). Same for labs.fpo and arm.fpo.

      h) Verify all paths and links on http://spins.fpo, labs.fpo and arm.fpo.

      i) Update Alpha Image sizes and pre_cloud_composedate in ./build.d/globalvar.py.
         Verify they are right in Cloud images and Docker image.

      j) Update the new POT files and push them to Zanata (ask a maintainer to do
         so) every time you change text strings.

      k) Add this build to stg.fedoraproject.org (ansible syncStatic.sh.stg) to
         test the pages online.

      l) Release Date:
        * Merge the fXX-alpha branch to master and correct conflicts manually
        * Remove the redirect of prerelease pages in ansible, edit:
        * ansible/playbooks/include/proxies-redirects.yml
        * ask a sysadmin-main to run playbook
        * When ready and about 90 minutes before Release Time push to master
        * Tag the commit as new release and push it too:
          $ git tag -a FXX-Alpha -m 'Releasing Fedora XX Alpha'
          $ git push --tags
        * If needed follow "Fire in the hole" below.


    2.2 For Beta

      a) Create the fXX-beta branch from master
         fedora-web git:(master) git push origin master:refs/heads/f22-beta

         and checkout to the new branch:
         fedora-web git:(master) git checkout -t -b f22-beta origin/f22-beta

      b) Update the global variables
         Change curr_state to Beta for all arches

      c) Add Alpha banner
         Upload the FXX-Beta banner to static/images/banners/f22beta.png
         which should appear in every ${PRODUCT}/download/index.html page.
         Make sure the banner is shown in all sidebars, also in labs, spins, and arm.

      d) Check all Download links and paths in ${PRODUCT}/prerelease/index.html
         You can find all paths in bapp01 (sudo su - mirrormanager first) or
         you can look at the downlaod page http://dl.fedoraproject.org/pub/alt/stage

      e) Add CHECKSUM files to static/checksums and verify that the paths are
         correct. The files should be in sundries and you can query them with:
         $ find /pub/fedora/linux/releases/test/17-Beta/ -type f -name \
         *CHECKSUM* -exec cp '{}' . \;
         Remember to add the right checksums to the right websites (same path).

      f) Add EC2 AMI IDs for Beta. All IDs now are in the globalvar.py file.
         We get all data from there, even the redirect path to trac the AMI IDs.
         We now also have a script which is useful to get all the AMI IDs uploaded
         with fedimg. Execute it to get the latest uploads, but don't run the script too
         early, as new builds are added constantly.
         fedora-web git:(fXX-beta) python ~/fedora-web/tools/get_ami.py

      g) Add CHECKSUM files also to http://spins.fedoraproject.org in
         static/checksums. Verify the paths are correct in data/content/verify.html.
         (see point e) to query them on sundries01). Same for labs.fpo and arm.fpo.

      h) Remove static/checksums/Fedora-XX-Alpha* in all websites.

      i) Verify all paths and links on http://spins.fpo, labs.fpo and arm.fpo.

      j) Update Beta Image sizes and pre_cloud_composedate in ./build.d/globalvar.py.
         Verify they are right in Cloud images and Docker image.

      k) Update the new POT files and push them to Zanata (ask a maintainer to do
         so) every time you change text strings.

      l) Add this build to stg.fedoraproject.org (ansible syncStatic.sh.stg) to
         test the pages online.

      m) Release Date:
        * Merge the fXX-beta branch to master and correct conflicts manually
        * When ready and about 90 minutes before Release Time push to master
        * Tag the commit as new release and push it too:
          $ git tag -a FXX-Beta -m 'Releasing Fedora XX Beta'
          $ git push --tags
        * If needed follow "Fire in the hole" below.


    2.3 For GA

      a) Create the fXX branch from master
         fedora-web git:(master) git push origin master:refs/heads/f22

         and checkout to the new branch:
         fedora-web git:(master) git checkout -t -b f22 origin/f22

      b) Update the global variables
         Change curr_state for all arches

      c) Check all Download links and paths in ${PRODUCT}/download/index.html
         You can find all paths in bapp01 (sudo su - mirrormanager first) or
         you can look at the downlaod page http://dl.fedoraproject.org/pub/alt/stage

      d) Add CHECKSUM files to static/checksums and verify that the paths are
         correct. The files should be in sundries01 and you can query them with:
         $ find /pub/fedora/linux/releases/17/ -type f -name \
         *CHECKSUM* -exec cp '{}' . \;
         Remember to add the right checksums to the right websites (same path).

      e) At some point freeze translations. Add an empty PO_FREEZE file to every
         website's directory you want to freeze.

      f) Add EC2 AMI IDs for GA. All IDs now are in the globalvar.py file.
         We get all data from there, even the redirect path to trac the AMI IDs.
         We now also have a script which is useful to get all the AMI IDs uploaded
         with fedimg. Execute it to get the latest uploads, but don't run the script too
         early, as new builds are added constantly.
         fedora-web git:(fXX) python ~/fedora-web/tools/get_ami.py

      g) Add CHECKSUM files also to http://spins.fedoraproject.org in
         static/checksums. Verify the paths are correct in data/content/verify.html.
         (see point e) to query them on sundries01). Same for labs.fpo and arm.fpo.

      h) Remove static/checksums/Fedora-XX-Beta* in all websites.

      i) Verify all paths and links on http://spins.fpo, labs.fpo and arm.fpo.

      j) Update GA Image sizes and cloud_composedate in ./build.d/globalvar.py.
         Verify they are right in Cloud images and Docker image.

      k) Update static/js/checksum.js and check if the paths and checksum still match.

      l) Update the new POT files and push them to Zanata (ask a maintainer to do
         so) every time you change text strings.

      m) Add this build to stg.fedoraproject.org (ansible syncStatic.sh.stg) to
         test the pages online.

      n) Release Date:
        * Merge the fXX-beta branch to master and correct conflicts manually
        * Add the redirect of prerelease pages in ansible, edit:
        * ansible/playbooks/include/proxies-redirects.yml
        * ask a sysadmin-main to run playbook
        * Unfreeze translations by deleting the PO_FREEZE files
        * When ready and about 90 minutes before Release Time push to master
        * Update the short links for the Cloud Images for 'Fedora XX', 'Fedora
          XX-1' and 'Latest'
        * Tag the commit as new release and push it too:
          $ git tag -a FXX -m 'Releasing Fedora XX'
          $ git push --tags
        * If needed follow "Fire in the hole" below.


  3. Fire in the hole

   We now use ansible for everything, and normally use a regular build to make
   the websites live. If something is not happening as expected, you should get in
   contact with a sysadmin-main to run the ansible playbook again.

   All our puppet stuff, such as SyncStatic.sh and SyncTranslation.sh scripts are now
   also in ansible!

   Staging server app02 and production server bapp01 do not exist anymore, now our staging
   websites are on sundries01.stg and the production on sundries01. Change your scripts
   accordingly and as sysadmin-web you should have access to those servers as before.   


  4. Tips

    4.1 Merging branches

    Suggested by Ricky
    This can be useful if you're *sure* all new changes on devel branch should go into
    the master branch. Conflicts will be solved directly accepting only the changes
    in the devel branch.
    If you're not 100% sure do a normal merge and fix conflicts manually!

    $ git merge f22-beta
    $ git checkout --theirs f22-beta [list of conflicting po files]
    $ git commit 
