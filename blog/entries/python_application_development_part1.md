Title: Python Application Development (Part 1 of ??)
Author: Tom Willis
published_date: 2013-10-12
teaser: <p>Sometimes,  virtualenv isn't enough</p>

Python has a rich ecosystem of libraries distributed through
[pypi](https://pypi.python.org/pypi). The work flow of developing
libraries is pretty well covered out there. Typically you are
encouraged to use [virtualenv](http://www.virtualenv.org/en/latest/),
and you absolutely should. Mainly for isolating your library and it's
dependencies from other libraries you may be working on and the
dependencies that your OS and other software depends on.

The combination of using
[virtualenv](http://www.virtualenv.org/en/latest/) and developing your
library as an [egg](http://pythonhosted.org/setuptools/formats.html)
insures that the egg can be installed in a consistent way. That's
a good thing. If your library is an egg, then you can distribute it
through [pypi](https://pypi.python.org/pypi) and other libraries can
depend on it.

However, what I'm going to refer to as "python applications", can
benefit from a bit more tooling. And what I would like to cover is
some of the things that can help you development full blown python
applications.

What I would call a python application is a program that could be made
up of many libraries. Some may be already distributed through
[pypi](https://pypi.python.org/pypi), others may be developed
in-house. A python application will likely depend on various servers,
scripts and configuration information. The configuration information
may differ by environment.

In order to illustrate these ideas, I will hopefully walk you through
the development of a website from step 1. It's important to note that
the libraries I use for the website are not what I want to emphasize
here. It's the process of developing an application that can be
setup/developed in a consistent/reliable way.


## The application

For this article we will be building a web application that will
display information about our music collection (I assume that in 2013
there should be at least some music on your computer). This may not be
a terribly useful application and I wouldn't expect anyone to start a
profitable business based on it. It's merely a platform to use to get
some ideas across.


## Getting Started

We need to decide on a project structure eventually, but even before
that we need to create a repository for the application.

### Creating the repository(git)

    :::bash
   
	user@puter:~/projects$ mkdir music_info_proj/
	user@puter:~/projects$ cd music_info_proj/
	user@puter:~/projects/music_info_proj$ git init
	Initialized empty Git repository in /home/user/projects/music_info_proj/.git/
	(master #) user@puter:~/projects/music_info_proj$ 



We now have a clean git repository for our application. However as you
can see from my shell, we are on the master branch. Though this would
be fine for a personal project with no planned releases, let's pretend
we're going to have to do some formal release
cycle. [GitFlow](http://nvie.com/posts/a-successful-git-branching-model/)
is a nice work flow for managing changes and releases, and if you are
not familiar with it, spend some time getting familiar with it. It
really helps coordinate changes when you have to work on a team. 

### Creating the Dev branch

One of the ideas behind GitFlow is that the master branch should only
contain stable releases. Since we are just getting started we have no
releases much less stable ones. So let's create a dev branch. And all
our changes will go there. When we have a good release we'll merge it
to master and tag.

But before we can create a branch we need to commit something(not sure
if this is really true)


    :::bash
	
	(master #) user@puter:~/projects/music_info_proj$ touch .gitignore
	(master #) user@puter:~/projects/music_info_proj$
	git add .gitignore
	(master #) user@puter:~/projects/music_info_proj$ git commit -m "initial commit"
	[master (root-commit) 12e8a21] initial commit
	 0 files changed
	 create mode 100644 .gitignore
	(master) user@puter:~/projects/music_info_proj$ git branch dev master
	(master) user@puter:~/projects/music_info_proj$ git checkout dev
	Switched to branch 'dev'
	(dev) user@puter:~/projects/music_info_proj$ 



### Creating the virtualenv

As I mentioned above, developing your python library within a
virtualenv is always a good idea, and it's a good idea for python
applications as well. So let's go ahead and do that.


	:::bash

	(dev) user@puter:~/projects/music_info_proj$ virtualenv -p /usr/bin/python2.7 .env
	Running virtualenv with interpreter /usr/bin/python2.7
	New python executable in .env/bin/python2.7
	Also creating executable in .env/bin/python
	Installing Setuptools............................................................done.
	Installing Pip...................................................................done.
	(dev) user@puter:~/projects/music_info_proj$ 


Now that we've installed it we're off to a good start, however,
there's a problem. The virtualenv is not something we want to track in
our repository. So we'll have to add tell git to ignore it.


So we edit .gitignore like so...

    :::diff

	diff --git a/.gitignore b/.gitignore
	index e69de29..4c49bd7 100644
	--- a/.gitignore
	+++ b/.gitignore
	@@ -0,0 +1 @@
	+.env
    
So now that we won't be checking in our virtualenv by mistake, we have
another issue we need to address. Right now, if we were lucky enough
to have a team mate join to help us out, we would have to instruct
them in all the things they need to do in order to help develop the
application. So we will probably have to do some form of documentation
but, if you are like me, you would rather code than document. And the
more you can do automatically the less you need to document.

So far we would have to instruct our team mate to create a virtualenv
at .env inside the project directory after they have clones the
repository. What if instead we just instructed them to do a command
that did everything for them?

### The README for the developers

One such tool that can be used for things that may need to be done
like setting up the dev environment would be
[fabric](http://docs.fabfile.org/en/1.8/). I'm not going to go into
depth about how to use it. We are simply going to author a fabfile.py
and keep that in the repository.  And then create a README.md to let
our team mates know they need to have fabric and virtualenv installed
and then what task to run in fabric to get setup.

So first the fabfile.py


	:::diff

	diff --git a/fabfile.py b/fabfile.py
	new file mode 100644
	index 0000000..5431ab8
	--- /dev/null
	+++ b/fabfile.py
	@@ -0,0 +1,19 @@
	+from fabric.api import local
	+from fabric.state import env
	+import os
	+
	+
	+def get_setting(key, default=None):
	+    result = env.get(key, os.environ.get(key, default))
	+    if key not in env:
	+        env[key] = result
	+    return result
	+
	+
	+VENV_PATH = get_setting("VENV_PATH", "./.env")
	+ROOT_PYTHON = get_setting("ROOT_PYTHON", "/usr/bin/python2.7")
	+VENV_CMD = get_setting("VENV_CMD", "virtualenv")
	+
	+
	+def virtual_env(root_python=ROOT_PYTHON, venv_path=VENV_PATH, venv_cmd=VENV_CMD):
	+    local("%s -p %s %s" % (venv_cmd, root_python, venv_path))



With the fabfile.py written we can create a virtualenv like so...


    :::shell

	(dev *+) user@puter:~/projects/music_info_proj$ fab virtual_env
	[localhost] local: virtualenv -p /usr/bin/python2.7 ./.env
	Running virtualenv with interpreter /usr/bin/python2.7
	New python executable in ./.env/bin/python2.7
	Not overwriting existing python script ./.env/bin/python (you must use ./.env/bin/python2.7)
	Installing Setuptools.............................................................done.
	Installing Pip....................................................................done.

	Done.
	(dev *+) user@puter:~/projects/music_info_proj$ 


So now let's write the README.md


    :::diff
	  
	diff --git a/README.md b/README.md
	new file mode 100644
	index 0000000..d6bf851
	--- /dev/null
	+++ b/README.md
	@@ -0,0 +1,17 @@
	+# Setting things up for development
	+
	+## Prerequisites
	+
	+Make sure you have the following installed and in your path.
	+
	+* python2.7 at /usr/bin/python2.7
	+* [virtualenv](http://www.virtualenv.org/en/latest/)
	+* [fabric](http://docs.fabfile.org/en/1.8/)
	+
	+## Initializing
	+
	+Assuming all prerequisites are satisfied, simply run...
	+
	+    :::shell
	+
	+    user@puter:~/projects/music_info_proj$ fab virtual_env


Finally, we have some things to add to .gitignore that will help us
keep unwanted files out of the repository, mainly backup files and
compiled python files...

    :::diff

    diff --git a/.gitignore b/.gitignore
    index e69de29..fbe4494 100644
    --- a/.gitignore
    +++ b/.gitignore
    @@ -0,0 +1,3 @@
    +.env
    +*.pyc
    +*~


And now we have what could be a good stopping point for the
moment. We've made it possible for someone to clone the repository and
have a virtualenv created from python2.7 in the place where we expect
it to be and documentation for our team mates. Let's commit

	:::shell

	(dev *+) user@puter:~/projects/music_info_proj$ git add README.md
	(dev *+) user@puter:~/projects/music_info_proj$ git add fabfile.py
	(dev *+) user@puter:~/projects/music_info_proj$ git add .gitignore
	(dev +) user@puter:~/projects/music_info_proj$ git commit -m "virtualenv setup"
	[dev d5fb7d1] virtualenv setup
	 3 files changed, 39 insertions(+)
	 create mode 100644 README.md
	 create mode 100644 fabfile.py
	(dev) user@puter:~/projects/music_info_proj$ 

