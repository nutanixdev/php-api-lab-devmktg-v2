.. title:: Nutanix API App Lab

.. toctree::
  :maxdepth: 2
  :caption:     Required Labs
  :name: _req-labs
  :hidden:

  .. example/index
  api_app/intro
  api_app/setup
  api_app/structure
  api_app/external
  api_app/classes
  api_app/styling
  api_app/wiring
  api_app/javascript
  api_app/thoughts

.. _getting_started:

Welcome
#######

Welcome to Nutanix API Getting Started Lab (PHP) - v1.1.

Getting Started
###############

The Nutanix API App Labs will cover a couple of key points.

- Creation of a simple application based on the Laravel PHP framework.
- Creation of relevant views to display Prism information to the user.
- Backend models and controllers to carry out the "real" work i.e. leverage the Nutanix REST APIs.
- Scripts to create the interface between the front- and back-end parts of the application.

.. note::

  Because this lab is written to be completed by those with little to no experience with Laravel/PHP or Nutanix APIs, it will step-through each part of the application being created.  Experienced developers may wish to go over the API intro in the next section so that it can be integrated into their own applications.

.. _requirements:

Requirements
############

To successfully complete this app lab, you will need an environment that meets the following specifications.

- Git, either by command-line or GUI application
- An installation of PHP version 5.5.9 or later.  For this lab, we will use the built-in PHP web server; there's no requirement to install WAMP, MAMP, IIS etc.
- PHP Composer (install from the Composer_ website)
- The text editor of your choice

.. _Composer: https://getcomposer.org/doc/00-intro.mdf

.. _optional_components:

Optional Components
###################

In addition to the requirement components above, the following things are "nice to have".  They are not mandatory for these labs.

- A Github account.  This can be created by signing up directly through GitHub_.
- The GitHub Desktop_ application (available for Windows and Mac only)
- Previous experience with PHP or related scripting/web technologies
- Experience with the Laravel PHP framework
- Postman_, one of the most popular API testing tools available.

.. _GitHub: https://github.com/
.. _Desktop: https://desktop.github.com/
.. _Postman: https://www.getpostman.com/

.. _cluster_details:

Cluster Details
###############

This lab can be run in a couple of different ways.  Primarily:

- In an instructor-led environment, typically completed using Nutanix Frame.  These sessions can be arranged with Nutanix directly, if required.
- Self-paced, where access to a Nutanix cluster is the responsibility of the reader.