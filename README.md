Artifactorial
=============

Artifactorial is a web application that allows one to easily upload files on
the server. The application will help you to control the users that can view
and upload files and the available pseudo directories.


Features
========

Artifactorial is managing four kinds of entities:

 * artifacts
 * directories
 * users and tokens
 * links


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


Links
-----

Every User can create links that point to artifacts that they can read. This
links can then be shared with other users to give them access to the files.


Managing
========

Installing
----------

Artifactorial is a python application, based on *Django*.
In order to setup an Artifactorial server from the sources, run:

    # Create a directory
    mkdir website
    cd website
    # Grab the sources
    git clone https://github.com/ivoire/Artifactorial
    # Install the python dependencies
    virtualenv venv
    source venv/bin/activate
    pip install -r Artifactorial/requirements.txt
    # Create the django project
    django-admin startproject myproject
    mv Artifactorial/* Artifactorial/.* myproject
    rmdir Artifactorial

Now you should configure the Django project by updating the settings in
**myproject/settings.py**:
 * add 'Artifactorial' to the **INSTALLED_APPS** list.
 * set **MEDIA_ROOT** to the directory where the artifacts will be stored
 * configure the database by updating the DATABASES field (see the [Django
documentation](https://docs.djangoproject.com/en/1.9/ref/settings/#databases))
 * import **Artifactorial.settings** in your project settings

In **myproject/urls.py** add the url for Artifactorial. You can look at
**tests/test_urls.py**.


When everything is setup, you should create the database and create the admin
account:

    python manage.py migrate
    python manage.py createsuperuser

For a developement server, run the Django mini-server:

    python manage runserver

For a production server, have a look at the official [Django
documentation](https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/modwsgi/).
You will have to also configure **DEBUG** and **ALLOW_HOST** variables.


Using Artifactorial
-------------------

To view the artifacts, just browse to the url where Artifactorial was installed:

    firefox http://example.com/artifacts/

To upload a file into an anonymous directory called */pub*, run:

    curl -F 'path=@path_to_the_file.ext' http://example.com/artifacts/pub/

To upload a file into the directory */home/debian* owned by the user *debian*,
you should provide a token as a proof of your identity:

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

    curl 'http://example.com/shares/123456789abcdef123456abcdef12345'

Artifactorial also provide a way to retrieve the hash of a given file by making
a HEAD request. The md5 hash of the file will be available in the *Content-MD5*
header.

    curl --head 'http://example.com/artifacts/home/debian/debian-sid.iso'

The server will return the full link to the ressource.


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

Every user can create a link that point to an artifacts that it can read.
Making a PUT request will return the new hash:

    curl -X PUT -d path=home/debian/debian-sid.iso 'http://example.com/shares/'

The server will return the link


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
