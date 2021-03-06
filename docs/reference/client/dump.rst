#######################
django CMS Cloud Client
#######################


.. warning:: this is just a dump from the old README.rst of aldryn-client. needs cleanup.
.. warning:: titles depth is wrong
.. warning:: don't repeat stuff from addons/Boilerplate section. here we should cover basics like login and link to the other sections.



.. _cloud-client-installation:

**********
Installing
**********

``pip install aldryn-client``


****************
Using the client
****************

After installing, you'll have a command-line tool called ``aldryn`` at your disposal. Running
it with no arguments will give you all possible commands.

Logging in
==========

Before using the client, you need to log in, call ``aldryn login`` and provide your username and password. This will
store a token in your local ``.netrc`` file that will be used for subsequent requests.


****
Apps
****

see :ref:`addon-packaging`



Settings
========

If you want to provide settings (and a nice form) for your app, you may add a file ``aldryn_config.py`` to the root of
your app (next to setup.py).

This file **must** contain a class named ``Form`` which **must** subclass ``aldryn_client.forms.BaseForm``.

The ``Form`` class may contain any number of form fields.

Available fields are:

* ``aldryn_client.forms.CharField``
* ``aldryn_client.forms.CheckboxField``
* ``aldryn_client.forms.SelectField``
* ``aldryn_client.forms.NumberField``
* ``aldryn_client.forms.StaticFileField``

All fields must provide a label as first argument and take a keyword argument named ``required`` to indicate whether
this field is required or not.

``CharField`` may optionally define ``max_length`` and ``min_length`` arguments. ``SelectField`` takes a list of tuples
(Django style) as required second argument). ``NumberField`` has optional ``min_value`` and ``max_value`` arguments.
``StaticFileField`` takes an optional ``extensions`` argument which is a list of valid file extensions.


Generating settings
-------------------

To generate settings, define a ``to_settings`` method which takes two arguments: The cleaned data of your form and a
dictionary of existing settings (which contains ``MIDDLEWARE_CLASSES``).

Add the settings you want on to the settings dictionary and return it.


Custom field validation
-----------------------

If you want to have custom field validation, subclass a field and overwrite it's ``clean`` method, which takes a single
argument (the value to clean) and should return a cleaned value or raise ``aldryn_client.forms.ValidationError`` with
a nice message as to why the validation failed.

Custom Runtime APIs
===================

CMS Cloud apps can use special APIs to inject content into templates (either in the head or at the end of the body) from
Python.

Those APIs are:

* ``cmscloud.template_api.registry.add_to_head`` to add content to the head.
* ``cmscloud.template_api.registry.add_to_tail`` to add content to the end of the body.


If you're using those APIs, you probably want to do so from a models.py file.


Commands
========

The client has two commands to work with your apps: ``app validate``, which checks the ``app.yaml`` config and
``app upload`` which uploads your app.


************
Boilerplates
************

Boilerplates are a set of default templates and staticfiles, optionally with initial data.

To get started, create a normal Django (CMS) project and develop your Boilerplate. When you're happy with how it looks
and optionally added some default content, you can transform this into a Boilerplate.

In your base template (or base templates), you **must** include the variable ``{{ TEMPLATE_API_REGISTRY.render_head }}``
in the head section of your HTML, and ``{{ TEMPLATE_API_REGISTRY.render_tail }}`` just before the closing ``</body>``
tag. Further, your base template **must** include the ``{% cms_toolbar %}`` template tag right after th opening
``<body>`` tag (make sure you ``{% load cms_tags %}`` first). Also, you **must** include two sekizai namespaces,
``{% render_block 'css' %}`` in your head section and ``{% render_block 'js' %}`` just before the
``{{ TEMPLATE_API_REGISTRY.render_tail }}`` variable.

Now add a ``boilderplate.yaml`` file to the root of your project (next to the ``static`` and ``templates`` folders).

The ``boilerplate.yaml`` **must** be present to upload a Boilerplate to the CMS Cloud!

It requires at least the following keys:

* ``name``: The name of your Boilerplate
* ``version``: The version of this Boilerplate (**must** be compatible with ``LooseVersion``)
* ``description``: A description of your Boilerplate
* ``author``: An object with the the following keys:
    * ``name``: Your name!
    * ``url``: URL to your website (optional)
* ``license``: An object with the following keys:
    * ``name``: Name of your license (eg BSD)
    * ``text``: Full text of the license (pro tip: use !literal-include <filename>)
* ``templates``: A list of tuples in the form of ``(template_path, verbose_name)``. The ``template_path`` is the path to
                 the template as used by Django. The verbose name is what users will see.


Including initial data
======================

To include initial data in your Boilerplate, add ``aldryn_client`` to your installed apps in your project and call
the management command ``aldryn_dumpdata <outfile> <language>``. ``<outfile>`` must be a file named ``data.yaml``
located next to your ``boilerplate.yaml`` file. ``<language>`` is the language code of the language you want to include
('en' is a good default choice). Only one language can be included.


Handling relations in plugins
-----------------------------

If your plugins include relationships to other models that need to be included, define a setting
``ALDRYN_DUMPDATA_FOLLOW`` which is a list of strings in the form of ``PluginName.relationship_field``.



Commands
========

The client has two commands to work with your Boilerplates: ``boilerplate validate``, which checks the
``boilerplate.yaml`` config and ``boilerplate upload`` which uploads your Boilerplate.



***************
Local File Sync
***************

You can sync your files locally using the ``sync`` command. This command optionally takes an argument
``--sitename=<sitename>`` to specify which site to sync. This argument must be set the first time you use the command,
on subsequent calls in the same directory, it will use the same site.

.. warning::

    This command will **delete** the folders ``static/`` and ``templates/`` locally if they exist.

******************
Packaging OS X App
******************

::

   #!/bin/bash
   VM_IP=192.168.3.73
   ssh kim@$VM_IP './deploy/make_app.sh'
   scp kim@$VM_IP:deploy/packages/AldrynCloud.dmg ~/Desktop/

Workflow: This script calls  make_app.sh on the vm, which updates the repo & then calls the deploy.sh from the repo.
In the end, the .dmg file gets copied to your local desktop
