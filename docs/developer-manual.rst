Developer manual
================

Read the :ref:`Configuration` section first to understand which hooks
both integrators and developers can use to customize and extend Kotti.

.. contents::

Screencast tutorial
-------------------

Here's a screencast that guides you through the process of creating a
simple Kotti add-on for visitor comments:

.. raw:: html

   <iframe width="640" height="480" src="http://www.youtube-nocookie.com/embed/GC3tw6Tli54?rel=0" frameborder="0" allowfullscreen></iframe>

.. _content-types:

Content types
-------------

Defining your own content types is easy.  The implementation of the
Document content type serves as an example here:

.. code-block:: python

  from kotti.resources import Content

  class Document(Content):
      id = Column(Integer(), ForeignKey('contents.id'), primary_key=True)
      body = Column(UnicodeText())
      mime_type = Column(String(30))

      type_info = Content.type_info.copy(
          name=u'Document',
          title=_(u'Document'),
          add_view=u'add_document',
          addable_to=[u'Document'],
          )

      def __init__(self, body=u"", mime_type='text/html', **kwargs):
          super(Document, self).__init__(**kwargs)
          self.body = body
          self.mime_type = mime_type

You can configure the list of active content types in Kotti by
modifying the :ref:`kotti.available_types` setting.

Add views, subscribers and more
-------------------------------

:ref:`kotti.includes` allows you to hook ``includeme`` functions that
you can use to add views, subscribers, and configure other aspects of
Kotti and more.  An ``includeme`` function takes the *Pyramid
Configurator API* object as its sole argument.

Here's an example that'll override the default view for Files:

.. code-block:: python

  def my_file_view(request):
      return {...}

  def includeme(config):
      config.add_view(
          my_file_view,
          name='view',
          permission='view',
          context=File,
          )

To find out more about views and view registrations, please refer to
the `Pyramid documentation`_.

By adding the *dotted name string* of your ``includeme`` function to
the :ref:`kotti.includes` setting, you ask Kotti to call it on
application start-up.  An example:

.. code-block:: ini

  kotti.includes = mypackage.views.includeme

.. _Pyramid documentation: http://docs.pylonsproject.org/projects/pyramid/en/latest/

Working with content objects
----------------------------

.. include:: ../kotti/tests/nodes.txt
  :start-after: # end of setup
  :end-before: # start of teardown

.. _slots:

:mod:`kotti.views.slots`
------------------------

.. automodule:: kotti.views.slots

:mod:`kotti.events`
-------------------

.. automodule:: kotti.events

``kotti.configurators``
-----------------------

Requiring users of your package to set all the configuration settings
by hand in the Paste Deploy INI file is not ideal.  That's why Kotti
includes a configuration variable through which extending packages can
set all other INI settings through Python.  Here's an example of a
function that programmatically modified ``kotti.base_includes`` and
``kotti_principals`` which would otherwise be configured by hand in
the INI file:

.. code-block:: python

  # in mypackage/__init__.py
  def kotti_configure(config):
      config['kotti.base_includes'] += ' mypackage.views'
      config['kotti.principals'] = 'mypackage.security.principals'

And this is how your users would hook it up in their INI file:

.. code-block:: ini
  
  kotti.configurators = mypackage.kotti_configure

.. _develop-security:

Security
--------

Kotti uses `Pyramid's security API`_, most notably its support
`inherited access control lists`_ support.  On top of that, Kotti
defines *roles* and *groups* support: Users may be collected in
groups, and groups may be given roles, which in turn define
permissions.

The site root's ACL defines the default mapping of roles to their
permissions:

.. code-block:: python

  root.__acl__ == [
      ['Allow', 'system.Everyone', ['view']],
      ['Allow', 'role:viewer', ['view']],
      ['Allow', 'role:editor', ['view', 'add', 'edit']],
      ['Allow', 'role:owner', ['view', 'add', 'edit', 'manage']],
      ]

Every Node object has an ``__acl__`` attribute, allowing the
definition of localized row-level security.

The :func:`kotti.security.set_groups` function allows assigning roles
and groups to users in a given context.
:func:`kotti.security.list_groups` allows one to list the groups of a
given user.  You may also set the list of groups globally on principal
objects, which are of type :class:`kotti.security.Principal`.

Kotti delegates adding, deleting and search of user objects to an
interface it calls :class:`kotti.security.AbstractPrincipals`.  You
can configure Kotti to use a different ``Principals`` implementation
by pointing the ``kotti.principals_factory`` configuration setting to
a different factory.  The default setting here is:

.. code-block:: ini

  kotti.principals_factory = kotti.security.principals_factory

API
---

.. toctree::

   api.rst


.. _Pyramid's security API: http://docs.pylonsproject.org/projects/pyramid/dev/api/security.html
.. _inherited access control lists: http://www.pylonsproject.org/projects/pyramid/dev/narr/security.html#acl-inheritance-and-location-awareness
