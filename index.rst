.. title:: Nutanix API App Lab v2

.. toctree::
  :maxdepth: 2
  :caption:     Required Labs
  :name: _req-labs
  :hidden:

  .. example/index
  contents/master_follow
  contents/intro
  contents/setup
  contents/structure
  contents/external
  contents/classes
  contents/styling
  contents/wiring
  contents/javascript
  contents/thoughts

.. _getting_started:

Welcome
#######

Welcome to Nutanix API Getting Started Lab (PHP) - v2

Getting Started
###############

The Nutanix API App Labs will cover a couple of key points.

- Creation of a simple application based on the Laravel PHP framework.
- Creation of relevant views to display Prism information to the user.
- Backend models and controllers to carry out the "real" work i.e. leverage the Nutanix REST APIs.
- Scripts to create the interface between the front- and back-end parts of the application.

.. note::

  Because this lab is written to be completed by those with little to no experience with Laravel/PHP or Nutanix APIs, it will step-through each part of the application being created.  Experienced developers may wish to go over the API intro in the next section so that it can be integrated into their own applications.

  This version of the lab is also an update to v1.1 published in 2018.  Laravel 5.1 was used in that version, whereas this version will use Laravel 6.

.. _requirements:

Requirements
############

To successfully complete this app lab, you will need an environment that meets the following specifications.

- An installation of `PHP version 7.2 <https://www.php.net/manual/en/install.php>`_ or later.  For this lab, we will use the built-in PHP web server; there's no requirement to install WAMP, MAMP, IIS etc.  To check your PHP version, open a terminal or command prompt and enter the following command:

   .. code-block:: bash

      php --version

- `PHP Composer <https://getcomposer.org/doc/00-intro.md>`_.  Install from the `PHP Composer <https://getcomposer.org/doc/00-intro.md>`_ website, making sure you can run **composer** in the terminal after installation.

   .. code-block:: bash
  
      composer

- The code editor of your choice.  We recommend `Microsoft Visual Studio Code <https://code.visualstudio.com/>`_ or `Sublime Text 3 <https://www.sublimetext.com/>`_.
- A Nutanix cluster, running Acropolis 5.11 or later.
- An instance of Prism Central version 5.11 or later, with your lab cluster registered against that instance.

.. note::

   If your laptop or workstation is not yet setup for development, you may wish to complete the Nutanix Developer Marketing `Dev Environment Setup 1.0 Lab <https://nutanix.handsonworkshops.com/workshops/da53e510-16f4-4fea-9c48-546e70e51a6b/start/>`_, first.

Optional Components
###################

In addition to the requirement components above, the following things are "nice to have".  They are not mandatory for these labs.

- A Github account.  This can be created by signing up directly through `GitHub <https://github.com>`_.
- The `GitHub Desktop <https://desktop.github.com/>`_ application (available for Windows and Mac only).
- Previous experience with PHP or related scripting/web technologies
- Experience with the Laravel PHP framework
- `Postman <https://www.getpostman.com/>`_, one of the most popular API testing tools available.

Cluster Details
###############

This lab can be run in a couple of different ways.  Primarily:

- In an instructor-led environment, typically completed using Nutanix Frame.  These sessions can be arranged with Nutanix directly, if required.
- Self-paced, where access to a Nutanix cluster is the responsibility of the reader.

Get Started
###########

With all that out of the way, let's now start by looking at some of the `conventions that will be used <contents/master_follow/>`_.
