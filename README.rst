========
zkaffold
========

A plone product to install content on a plone site.

Features
--------

 * Install content

 * Install products

 * Delete content

 * Apply zope interfaces

 * Modify content

 * Generate test content

How to install
--------------

Add zkaffold to your product:

  * Add "zkaffold" to your product's install_requires in setup.py

  * Add <include package="zkaffold" /> in your product's configure.zcml

  * Add "profile-zkaffold:default" as a dependency to your product's
    metadata.xml

  * run ./bin/buildout

zkaffold should now be installed.  Note that zkaffold is not installable /
uninstallable from the portal quick installer.

How to use
----------

After installing zkaffold in your plone site, you can build content for
your plone site by:

 * create "initial" profile in your product,
 
 * create a directory called "zkaffold" in profile/initial,

 * create structure.xml (example structure.xml is in
   zkaffold/exportimport/tests/test_exportimport.py),

You can also create default users in your plone site by:

 * create members.xml (example members.xml is in
   zkaffold/exportimport/tests/test_exportimport.py)

If you are using buildout, in your buildout.cfg:

 * in the [plonesite] section, add your product initial profile (e.g.
   mysite:initial) to "profiles-initial",
 * run ./bin/buildout

Your plone site should now be populated with content specified in
structure.xml.  Note that if your plone site already exists before running
buildout, it will not add any content.  You'll have to delete your plone
site or run the zkaffold import step to add the content.

You can use different profiles to create different content, for example if
you want test content, you can create another profile and add that in
profiles-initial when you want test content to be built.

zkaffold publishes all objects by default by trying to apply transition
"publish" to the objects.  If your objects need a different way of
publishing, have a look at zkaffold/exportimport/tests/test_exportimport.py.

zkaffold XML schema
-------------------
The basic schema that all zkaffold XML files should begin with is::

  <?xml version="1.0" encoding="UTF-8"?>
  <site xmlns:zkaffold="http://www.isotoma.com/zkaffold">
  </site>

The ``site`` element is the lone root element and represents the Plone portal
object.

Descending the object tree
``````````````````````````
Objects may be traversed by their ID in order to reach the object which should
be modified. This is done by specifying elements with the tag name as the ID
of the object in question. For example, to descend to the object at path
"``portal/my_folder/my_obj``" , one would use the following syntax::
  
  <site>
    <my_folder>
      <my_obj>
        ...
      </my_obj>
    </my_folder>
  </site>

Adding new objects
``````````````````
To insert a new object into a folderish object, first descend to the target
container, then insert a new element with the same tag name as the portal type
you wish to add, with an ``id`` attribute of the ID it should have within the
ZODB::
  
  <site>
    <my_folder>
      <MyContentType id="my_new_obj">
      </MyContentType>
    </my_folder>
  </site>

Parameters
''''''''''
More often than not, one will want to specify values for fields in the schema
of the object when it is created. To do this, add a ``params`` child to the
element representing the new object, which itself should contain a ``param``
element for each field. The ``param`` tag should have the ``name`` attribute
set to the name of the field in question and the ``type`` attribute set to one
of ``text``, ``lipsum``, ``lipsum_html``, ``reference``, ``image``, ``file``,
``boolean``, ``list`` or ``reference_list``. The ``param`` element then
contains the value that should be given to that field. One need not specify a
``param`` element for each and every field of a content type, the default
values will be used where fields are not specified.

``param`` elements with the ``type`` attribute set to ``lipsum`` or
``lipsum_html`` have the optional attribute ``paras`` which specifies the
number of paragraphs of `Lipsum <http://en.wikipedia.org/wiki/Lorem_ipsum>`_
to generate as the value of that field. They also contain no child elements.
The ``lipsum`` value causes ordinary Lipsum to be generated with newlines,
whereas ``lipsum_html`` generates Lipsum with paragraph tags as necessary::

  <site>
    <my_folder>
      <MyContentType id="my_new_obj">
        <params>
          <param name="title" type="text">My new object</param>
          <param name="text" type="lipsum_html" paras="5" />
          <param name="isNew" type="boolean">True</param>
          <param name="old_obj" type="reference">/my_folder/my_obj</param>
          <param name="thumb" type="image">new_obj_thumb.jpg</param>
        </params>
      </MyContentType>
    </my_folder>
  </site>

As in the above example, references are given as path strings to the
referenced object. Images & files are given as file names relative to the
directory of the XML file.

Multi-valued parameters
'''''''''''''''''''''''
For fields which may contain multiple values such as ``ReferenceList`` or
``LinesField``, the ``param`` element takes the ``type`` attribute ``list`` or
``reference_list``. It should then contain an ``items`` element, which
contains zero or more ``item`` elements with their ``type`` attribute set
appropriately, as per the ``param`` tag and their value set as their inner
XML::

  <site>
    <my_folder>
      <MyContentType id="my_new_obj">
        <params>
          <param name="title" type="text">My new object</param>
          <param name="options" type="list">
            <items>
              <item type="text">Option 1</item>
              <item type="text">Option 2</item>
            </items>
          </param>
        </params>
      </MyContentType>
    </my_folder>
  </site>

Interfaces
''''''''''
Occasionally one may wish to add interfaces to objects which they do not
automatically provide, particularly marker interfaces. This is achieved using
the ``interfaces`` & ``interface`` tags, like so::

  <site>
    <my_folder>
      <MyContentType id="my_new_obj">
        <interfaces>
          <interface>identifier.of.IMyMarker</interface>
          <interface>identifier.of.IReplacementObject</interface>
        </interfaces>
        <params>
          <param name="title" type="text">My new object</param>
        </params>
      </MyContentType>
    </my_folder>
  </site>

This adds the interfaces specified by the identifiers to the object.

Interfaces that are already applied to objects may be similarly removed by
replacing the ``interface`` element with a ``remove`` element, thus::
  
  <site>
    <my_folder>
      <MyContentType id="my_new_obj">
        <interfaces>
          <remove>IMyMarker</remove>
        </interfaces>
        <params>
          <param name="title" type="text">My new object</param>
        </params>
      </MyContentType>
    </my_folder>
  </site>

Modifying existing objects
``````````````````````````
Existing objects may be modified in a very similar way to which they are
added. Simply descend to the object in question, append the ``modify`` element
to it, and add ``params`` and ``interfaces`` elements as its children, as
necessary. Instead of taking the default values where fields are not
specified, the fields will retain their current values::

  <site>
    <my_folder>
      <my_new_obj>
        <modify>
          <params>
            <param name="title" type="text">My updated title for my new object</param>
          </params>
        </modify>
      </my_new_obj>
    </my_folder>
  </site>

Setting the default page of folderish objects
'''''''''''''''''''''''''''''''''''''''''''''
To set the default page of a new or existing folderish object, use the
``modify`` element with a ``setDefaultPage`` attribute::

  <site>
    <my_folder>
      <modify setDefaultPage="my_new_obj">
        ...
      </modify>
    </my_folder>
    <FolderishType id="another_folder">
      <modify setDefaultPage="folder_home" />
      <params>
        ...
      </params>
      <ContentType id="folder_home">
        ...
      </ContentType>
    </FolderishType>
  </site>

Deleting objects
````````````````
Existing objects can be deleted by descending to their container object and
inserting a ``delete`` element with its ``id`` attribute set to the ID of the
object which should be deleted. The ``delete`` element also has the optional
parameter, ``ctype`` which can be used to specify a particular content type
which the specified object must be in order to be deleted::
  
  <site>
    <my_folder>
      <!-- I never liked that new object anyway... -->
      <delete id="my_new_obj" ctype="MyContentType" />
    </my_folder>
  </site>
 
Dependencies
------------

zkaffold depends on lxml, which depends on libxml2-dev and libxslt-dev.
In Debian, you can use:

 * ``sudo aptitude install libxml2-dev libxslt-dev``

PLUGIN
------

zkaffold supports a plugin system for exporting content:

 * Add an import step for your profile initial,

 * create an exporter for your field, it should return a
   lxml.etree.Element for <param> or (lxml.etree.Element for <param>,
   filename, file content)::

     def my_field_exporter(plone_object, field):
         ...
         return param

 * create the import step to register the field exporter::

     from zkaffold.contextexporter import IContentExporter

     def register_field_exporters(context):
         portal = context.getSite()
         sm = portal.getSiteManager()
         exporter = sm.getUtility(IContentExporter)
         exporter.register_field_exporter('mysite.fields.MyField',
             'mysite.field_exporters.my_field_exporter')

Tests
-----

To run zkaffold's tests, you need to:

 * add "zkaffold [test]" in the "eggs" in the [test] section of your
   buildout.cfg,

 * run ./bin/buildout,

 * ./bin/test
