Getting Started
===============

.. highlight:: sh

This project was created with Chef Solo in mind, so certain optimizations like
search are not utilized. These cookbooks should still run smoothly on Chef
client/server, though. They are primarily for Django sites, and should be
suitable for both small one-server projects or large multi-server deployments.

Currently the only OS supported is Ubuntu (Lucid or above). To use these
cookbooks, you will create some site-specific configuration files in your
project's source control. These will usually include nodes, roles, and a
site-cookbook for your site. These site-specific files will work with the
modular cookbooks here to fully provision your servers.

Contributions are welcome, so feel free to fork and send pull requests if you
have something you'd like to share.

First Steps
***********

To begin, create a folder called "config" under the top-level project directory
for your site, which should be under version control.  This will hold your
configuration files.

Installing Fabric
-----------------

An easy way to bootstrap your new server now, and run deploys on your server
later, is to use `Fabric`_ with the *fabfile.py* example included in this
project.  Fabric is a Python-based tool for automated control of one or more
servers via SSH.  If you don't already have it, use *easy_install* or *pip* to
install it::

    $ easy_install fabric
    
    # or
    
    $ pip install fabric

.. _Fabric: <http://docs.fabfile.org/>

Copy *fabfile.py.example* from this project, and remove the *.example* from the
name.  Save it to your project's root directory.  Edit the ``env.roledefs``
attribute to match your environment.

A basic ``env.roledefs`` which described 1 server would look like this:

.. code-block:: python

    env.roledefs = {
        'dev': ['myserver'],
    }

In this example, 'myserver' is the hostname of your remote machine.  This
fabfile defaults to the 'dev' environment if nothing is specified on the
command line.

Minimal Config
--------------

To start out with the most basic usable config, which will set up the server
and a user, first create a *nodes* directory inside *config*.  Within that
directory, create a *.json* file with the same name as your server hostname
(which you specified in ``env.roledefs`` if you're using the included fabfile).

Edit *myserver.json* to look like this, inserting your desired values:

.. code-block:: js

    {
      "run_list": ["server"],

      "users": {
        "myuser": {
          "full_name": "My User",
          "groups": ["admin"],
          "ssh_keys": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC/jsn4rNFm1Uy7tJZqCkXrQzKM8ih2fBqmrRV9kGNRRAF4mp91ueE2iEfTShpaB5vknV09yASu5DabhQAy5TPaUz0spaG8gYLwYjdNObeD9NMNSkTzeG71s2iPXzT9621lEuZRwWkQuT8/MJ46odi+xrbXVBMNnSkw/t41IOMfKmEhRo1T1+VL3ZBG4S76PKrwfOt9VDMs1m+ISDvr10OTk1jz+8W0PgNnduiAF6jRtiXqOtPbrnh7njn4CgwpxCZOKYya5qbuWfJOZE9vEFUR7C+aCr/OyrETCO9MPaHVoe4JPxw5PKaE3NdAOwUlDA4hnbnh7VCJcCwQgVKwgMsf myuser@example.com"
        }
      }
    }

The "ssh_keys" attribute will create the "authorized_keys" file so that you
can connect to your server without needing a password.  Use your dev machine's
public key for this.

Next, create a *solo.rb* file under the *config* directory.  This should look
similar to the following:

.. code-block:: ruby

    log_level :info
    cookbook_path ["/etc/chef/cookbooks", "/etc/chef/site-cookbooks"]
    role_path "/etc/chef/roles"

Bootstrapping
-------------

You are now ready to bootstrap your machine, which will install all the
packages needed for Chef and then sync your configuration.  Switch to the
*config* directory and run the ``fab bootstrap`` command::

    $ fab bootstrap

You'll likely have to enter your root password twice along the way, but at the
end of the process you should be able to SSH into your server under the user
you created without needing a password.

Syncing Config Changes
**********************

To update your configuration, you need to run the ``fab sync_config`` command.
This command will use *rsync* to synchronize your local *config* directory to
each remote server, and then run *chef-solo* to update the configuration::

    $ fab sync_config

Note that the files in your local *config* directory will be copied directly
to the server using *rsync*, so make sure you are on the right branch before
running the sync.

Syncing a Different Environment
-----------------------------------

Fabric will default to the "dev" environment. If you want to sync a
different environment, you will need to use the -R option and specify the
rolename from ``env.roledefs``. For example::

    $ fab -R prod sync_config

Syncing a Specific Host
----------------------------

If you need to sync to just one host, you can use the -H option. For example::

    $ fab -H myhost sync_config
