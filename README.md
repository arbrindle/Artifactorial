Artifactorial
=============

Artifactorial is a web application that allows one to easily upload files on
the server. The application will help you to control the users that can view
and upload files and the available pseudo directories.


Features
========

Artifactorial is managing three kind of entities:

 * artifacts
 * directories
 * users and tokens


Artifact
--------

An artifact is a file that as been uploaded into Artifactorial. Every artifact
belongs to one directory.

An artifact can be either permanent or temporary. By default an artifact is
temporary and thus will be automatically removed after some days in the
directory. This duration, called Time To Live (TTL), is specific to each
directory.

Directory
---------

A directory is a repository that contains artifacts. Every directory is owned by either:

 * a user: only the user can upload artifacts
 * a group: all the user in the group can upload artifacts
 * no one (anonymous): everyone can upload artifacts

A directory is either public or private. Making listing available to everyone
or only the owners.

Each directory size is limited by a quota. Uploading artifacts in a directory
is only possible if the quota allows the upload.

Every directory also have a Time To Live. This is the maximum number of days a
temporary artifact will stay in the directory, before behind deleted by the
cleaning process.


Users and Tokens
----------------

A user can upload and view artifacts in:

 * his directories
 * his groups directories
 * the public directories

In order to identify a user, Artifactorial require the user to provider his
*token*.

A *token* is just a random string that is generated by Artifactorial to
uniquely identify a user. Each user can have multiple tokens.
To get a *token*, ask an administrator to create one (or more) for you.


Managing
========

Installing
----------

Artifactorial is a python application, based on *Django*. In order to install the dependencies, run:

    pip install -r requirements.txt


Using Artifactorial
-------------------

To view the artifacts, just browse to the url where Artifactorial was installed:

    firefox http://example.com/artifacts/

To upload a file into a anonymous directory called */pub*, run:

    curl -F 'path=@path_to_the_file.ext' http://example.com/artifacts/pub/

To upload a file into the directory */home/debian* owned by the user *debian*, run:

    curl -F 'path=@debian-sid.iso' -F 'token=123456789' http://example.com/artifacts/home/debian/

When you upload a file, Artifactorial will return the URL that can be used to
download it back. If this artifact is in a private directory, you would have to
provide the token as a GET parameter like:

    curl 'http://example.com/artifacts/home/debian/private/debian-sid.qcow2?token=123456789'

To mark an artifact as *permanent*, just upload it with:

    curl -F 'path=@path_to_the_file.ext' -F 'is_permanent=1' http://example.com/artifacts/pub/

Programs can browse Artifactorial by using JSON and YAML outputs with:

    curl 'http://example.com/artifacts/home/?format=json'
    curl 'http://example.com/artifacts/home/?format=yaml'

It's also possible to create a link to share a specific artifact with someone
without any right on the directory that contains the artifact.
The artifact can then be downloaded using

    curl 'http://example.com/shared/123456789abcdef123456abcdef12345'

Administration
--------------

We advise you to run the *clean* command every day to remove files that are too
old (see the *TTL* value of the directories).
Running the command is a matter of:

    python manage.py clean

Once in a while it can be interesting to purge old artifacts (including
permanent ones): use the *purge* command:

    python manage.py purge --ttl time_to_live_in_days --all

Without the *--all* parameter, the *purge* command will only remove temporary
artifacts.


Admin interface
---------------
The Django automatic administration interface can be used to manage most
aspects of Artifactorial. In fact, this interface can be used to create, update
and delete:

 * directories
 * artifacts
 * users and token


Contributing
============

If you want to contribute to Artifactorial, please fork it on github and send
me some pull requests.
