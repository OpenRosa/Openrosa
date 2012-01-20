===============================
XForms as supported by JavaRosa
===============================

Introduction
============

  Note: this page assumes advanced proficiency with xforms and JavaRosa. If you're looking for information on how to write forms, please consult our `introductory guide <https://bitbucket.org/javarosa/javarosa/wiki/GettingStarted>`_.

The full XForms specification is large and complex. JavaRosa runs on mobile devices that have limited resources and non-standard UI. Consequently, we only support a limited subset of the xforms spec, and in some cases only support it in idiosyncratic ways. This page attempts to precisely document which portions of the xforms spec are supported in JavaRosa.

JavaRosa is an actively evolving application, so if a certain feature is not supported, that may simply be because no one has had time to implement it yet. Expect the JavaRosa xforms engine to grow ever-more powerful over time. Also, if there is a certain capability you want that is missing, notify the JavaRosa developers, or if you have the means, add it yourself!

However, given xforms' origins in rich web clients, and mission to support application-like user experience, certain xforms features are simply inappropriate for a mobile platform, and will not be implemented.

Furthermore, JavaRosa has a defined a number of additions/customizations to the xforms spec, which address shortcomings and improves the xforms experience on mobile devices. These custom constructs are supported only by JavaRosa, and have been created when no xforms idiom is able to handle our desired use case, or the 'proper' xforms way of doing it is too complicated and resource-intensive. When customizations have been added, we have attempted to keep them as 'xform-y' as possible.

 * `XForms 1.0 specification <http://www.w3.org/TR/xforms/>`_
 * `JavaRosa XForm Validator <https://www.commcarehq.org/formtranslate/#>`_ - a tool to test if your XForm is JavaRosa-compliant


Overall form structure 
======================

XForms elements are intended to be hosted inside some other XML schema. JavaRosa XForms typically use XHTML for this purpose. We enclose the XForms ``model`` inside the html ``<head>`` tag, and the XForms controls inside the html ``<body>`` tag. We also use the html ``<title>`` and ``<meta>`` tags to describe meta-data about the form, including form title, form ID/descriptor, and form version.

User controls
=============

XForms defines the following user controls:

+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|Control       |  Supported                                                                                                                                                          |
+==============+=====================================================================================================================================================================+
|``<input>``   |fully supported in JavaRosa for data types text, integer, decimal/fractional number, and date. other data types not yet supported                                    |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<select1>`` |supported in JavaRosa, except no support yet for dynamic choice lists (``<itemset>``),or 'open' selections (where you can manually enter in a choice not in the list)|
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<select>``  |supported in JavaRosa, except no support yet for dynamic choice lists (``<itemset>``),or 'open' selections (where you can manually enter in a choice not in the list)|
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<secret>``  |no support yet in JavaRosa. easy to implement but no compelling use case thus far                                                                                    |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<textarea>``|no support yet in JavaRosa. easy to implement but ``<input>`` with type text does just as well                                                                       |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<range>``   |no support yet in JavaRosa                                                                                                                                           |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<submit>``  |no support in JavaRosa, and probably never will be (form submission is handled at the application level)                                                             |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<upload>``  |limited support for image and audio capture; very application/use case specific. expect development on this in the future                                            |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<output>``  |limited support for specific use cases, but not generally usable. expect development on this in the future                                                           |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``<trigger>`` |extremely limited support for specific use cases, but not generally usable. unlikely that comprehensive support for this will be added                               |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Within the user controls, xforms defines the following descriptive tags:

+-------------+----------------------------------------------------------------------------+
| Tag         | Supported                                                                  |
+=============+============================================================================+
|``<label>``  |fully supported                                                             |
+-------------+----------------------------------------------------------------------------+
|``<hint>``   |parsed, but not currently used in the form entry UI                         |
+-------------+----------------------------------------------------------------------------+
|``<help>``   |no support currently                                                        |
+-------------+----------------------------------------------------------------------------+
|``<alert>``  |no support currently (I'm unclear on what situations this tag is even for)  |
+-------------+----------------------------------------------------------------------------+


You can alter the look and feel of an xforms control through the ``appearance`` attribute, but JavaRosa doesn't currently support this in any meaningful way.

Instance and Model
==================

JavaRosa does not currently support multiple ``<model>`` s or multiple ``<instance>`` s.

In the instance, you can bind questions only to leaf nodes. Instance nodes may contain attributes, and these attributes will be preserved when the form is submitted, but you cannot currently bind any question to an attribute (e.g., ``<bind nodeset="patient/@id" ... />`` is not supported) or access the value of an attribute in an expression.

The top-level instance node is recommended, but not required, to have an ``xmlns`` attribute identifying the schema of the form instance.

Bindings
========

There is no support for nested bindings.

XForms defines the following binding attributes:

+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|Binding        |Supported                                                                                                                                                                                                                                |
+===============+=========================================================================================================================================================================================================================================+
|``type``       |supported, though supported data types vary by control. namespace prefix for the type may be omitted (e.g., ``date`` will be accepted in addition to the formally correct ``xsd:date``)                                                  |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``readonly``   |fully supported, but see support caveats for xpath expressions                                                                                                                                                                           |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``required``   |fully supported, but see support caveats for xpath expressions                                                                                                                                                                           |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``relevant``   |fully supported, but see support caveats for xpath expressions                                                                                                                                                                           |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``constraint`` |fully supported, but see support caveats for xpath expressions. also supports a custom attribute ``jr:constraintMsg`` to provide a custom alert to the user when the constraint is violated. (maybe this is what ``<alert>`` is for?)    |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``calculate``  |no support yet                                                                                                                                                                                                                           |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``p3ptype``    |will never be supported                                                                                                                                                                                                                  |
+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

We also support two custom bind parameters, ``jr:preload`` and ``jr:preloadParams``, used for defaulting in data when the form is loaded. Normally you'd just hard-code the data in the instance to default it, but this isn't possible for **dynamic** data that you want to pre-load into your instance. Thus, we have these parameters to call out to various preloaders. A preloader is a Java handler that is registered with the xforms engine and provides certain kinds of data. The ``jr:preload`` parameter identifies which registered preloader to call out to; the ``jr:preloadParams`` parameter passes information to the preloader about what kind of data you want back.

Some built-in preloaders include:

+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
|Preloader        |Used for                                                                                                                                         |
+=================+=================================================================================================================================================+
|``date``         |used for getting dates, like today's date, yesterday, tomorrow, the 5th previous Wednesday, etc.                                                 |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
|``timestamp``    |used for getting timestamps, like when the form was opened, when it was completed                                                                |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
|``property``     |used for fetching data stored in JavaRosa properties, like device ID                                                                             |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
|``context``      |used for fetching data from the current application context, like logged-in user                                                                 |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
|``patient``      |used for retrieving data from a patient record, assuming a given patient has been associated with the form entry session at the application level|
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------+

The data returned by the preloader is stored into the instance as the default value. Contact a developer for more details on how to use preloaders. Note: the custom preload parameters may be deprecated in the future, and replaced with a scheme that uses ``<setvalue>`` and custom XPath functions.

Paths and Expressions
===============================

XPath paths are used in XForms to reference instance nodes to store or retrieve data. JavaRosa currently supports both absolute and relative paths, along with using the proper relative path context node, depending on the situation. Paths can currently only reference XML elements (not attributes, comments, or raw text). Paths can currently only use the parent and child axes, and cannot have predicates. Basically, this means only paths of the form ``/``, ``a-node``, ``/path/to/a/node``, and ``here/is/another/path`` are allowed. You may also use the shortcuts ``.`` and ``..`` at any point in the path.

Rules for the relative path context node are as follows:
 * for xforms controls at the top level (not inside a ``<group>`` or ``<repeat>``), and for ``nodeset`` s in ``<bind>`` s, the context node is the top-level instance node (the immediate and only child of ``<instance>``)
 * for xforms controls inside a ``<repeat>`` or ``<group>``, the context node is the binding node of the ``<group>`` or ``<repeat>``. If inside a ``<group>`` that does not define a binding, use the next ``<group>``/``<repeat>`` up the hierarchy, until the top level is reached
 * for expressions inside attributes in a ``<bind>``, the context node is the node defined in the ``nodeset`` attribute of that ``<bind>``.

Controls and ``<repeat>`` s may use the ``ref`` and ``nodeset`` attributes, respectively, to define their binding. They may also use the ``bind`` attribute to refer directly to a given ``<bind>`` element via its ``id`` attribute. There is not much point to this convention, though, as it just adds one more step of indirection. You should favor using ``ref`` and ``nodeset``.


XPath expression are supersets of XPath paths. An XPath expression contains various XPath paths, along with comparisons, operators, and functions, to evaluate some condition. These conditions are used for the ``relevant`` , ``constraint`` , and more rarely, ``readonly`` and ``required`` attributes. When evaluating, any paths are replaced with the data value currently stored in the node referenced by the path. If a path does not uniquely identify a single node, but instead a set of several nodes (such as a node that may be ``<repeat>`` ed), the path will not evaluate to a data value, but instead evaluates to a **nodeset** .

Consult the `XPath 1.0 specification <http://www.w3.org/TR/xpath>`_ for details on XPath. The JavaRosa implementation of XPath is quite feature complete, and follows proper XPath semantics very closely. The only exception is the set of built-in XPath functions. JavaRosa only supports a limited subset of XPath functions, and adds some custom ones as well.



JavaRosa XPath functions:

+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|Function                                      |Action                                                                                                                                                                                                                                                                                                                                       |
+==============================================+=============================================================================================================================================================================================================================================================================================================================================+
|``true()``                                    |returns true                                                                                                                                                                                                                                                                                                                                 |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``false()``                                   |returns false                                                                                                                                                                                                                                                                                                                                |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``boolean(x)``                                |converts ``x`` to a boolean value; conversion varies depending on data type of ``x``                                                                                                                                                                                                                                                         |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``number(x)``                                 |converts ``x`` to a numeric value; conversion varies depending on data type of ``x``                                                                                                                                                                                                                                                         |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``string(x)``                                 |converts ``x`` to a string value; conversion varies depending on data type of ``x``                                                                                                                                                                                                                                                          |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``date(x)``                                   |converts ``x`` to a date value; conversion varies depending on data type of ``x``                                                                                                                                                                                                                                                            |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``not(x)``                                    |negates boolean value ``x``                                                                                                                                                                                                                                                                                                                  |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``boolean-from-string(x)``                    |returns true if ``x`` is ``"true"`` or ``"1"``, false otherwise. note that this is different behavior than ``boolean(x)``                                                                                                                                                                                                                    |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``if(cond, a, b)``                            |evaluate ``cond``; if true, return ``a``, else, return ``b``                                                                                                                                                                                                                                                                                 |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``selected(multi-select, choice)``            |``multi-select`` is the path to the answer of a multi-select question; return true if ``choice`` is one of the selected answers                                                                                                                                                                                                              |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``count-selected(multi-select)``              |``multi-select`` is the path to the answer of a multi-select question; return the number of selected answers                                                                                                                                                                                                                                 |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``count(nodeset)``                            |count the number of nodes in ``nodeset``                                                                                                                                                                                                                                                                                                     |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``sum(nodeset)``                              |return the sum of the values of the nodes in ``nodeset``                                                                                                                                                                                                                                                                                     |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``today()``                                   |return today's date                                                                                                                                                                                                                                                                                                                          |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``now()``                                     |return a timestamp for this instant                                                                                                                                                                                                                                                                                                          |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|``checklist(min, max, v1, v2, v3, ..., vn)``  |``v1`` through ``vn`` are a set of n yes/no answers. return true if the count of 'yesses' is between ``min`` and ``max``, inclusive. ``min`` or ``max`` may each be ``-1`` to indicate 'not applicable'. this is how you do 'at least 3 of ...' and 'at most 4 of ...' -type conditions                                                      |
+----------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Additional functions may be added via custom Java function handlers that are registered at run-time.

Many operations are only defined for a certain set of data types. XPath typically makes a good faith effort at converting your value to the proper data types, but make sure you understand the semantics of the type conversion, or you may get confusing results. Sometimes, it is not possible to convert your value to the desired type, and you will get a run-time error. For example, using a nodeset anywhere other than where a nodeset is explicitly expected will usually cause a type error.

Groups and Repeats
===============================

JavaRosa supports ``<group>`` s. Groups may have a ``label``, a ``ref``, both, or neither. If a ``<group>`` has a ``<label>``, it is considered a presentation group, and visual hints will be presented to the user as they fill out the form that the questions within the group are related (such as placing all questions under a banner whose caption is taken from the ``<group>``'s ``<label>``). If a ``<group>`` contains a ``ref`` attribute, it is considered a logical group, meaning the answers for all the questions within the group are stored together ('grouped') in the instance (i.e., the ``<group>``'s ``ref`` is the parent of the ``ref`` s of all child questions in the ``<group>``). To that end, the ``<group>``'s ``ref`` is used as the context node for the relative ``ref`` paths of the group's child questions, allowing these ``ref`` s to be abbreviated even further. (note: it is not strictly required that child questions of a logical group store their answers under the ``<group>``'s ``ref`` in the instance (such questions may use an absolute path pointing elsewhere for their ``ref``), but it is confusing to do so, so please don't).

We also support the ``<repeat>`` tag, which allows repeating a question or set of questions an arbitrary number of times. The ``nodeset`` attribute of the ``<repeat>`` specifies the instance node to be parameterized -- that is, it is this exact node (and all its children) which is duplicated each time a new repetition is created. Our implementation of ``<repeat>`` is pretty standard, but it diverges in several ways.

In pure xforms, 'create new repetition' and 'delete repetition' actions are initiated manually by the user, who does so using special ``<trigger>`` controls created explicitly for this purpose. JavaRosa provides these actions automatically as part of the form entry UI. For example, the user will be prompted when necessary if they want to create a new repetition, and when they are inside a ``<repeat>``, the 'delete' option is available under a menu. These ``<trigger>`` s are unnecessary, and will be ignored if present.

In xforms, data preloaded into your instance is treated as default values. However, for ``<repeat>`` able nodes, it is ambiguous whether to use the data as the default for just the one repetition, or as the default for all future repetitions. To resolve this, we added the ``jr:template`` attribute. Without this custom attribute, every repetition of a node will be considered valid data, and the default values within each node will be used as default values only for that repetition; the default values for all newly-created repetitions will be blank.

Example::

   <instance>
     <data xmlns="...">
      <name>Jimmy</name>
      <name />
      <name>Julia</name>
     </data>
   </instance>

``<name>`` is a repeatable node. The form entry UI will treat this as having three repetitions already created. When going through the form, the user will see the question 'Name 1:' with a default of 'Jimmy', then 'Name 2:' with no default, then 'Name 3:' with a default of 'Julia'. After 'Name 3:', they will be asked 'Add another Name?'. If they do, the default for this new repeated ``name`` will be blank.

Now lets add the ``jr:template`` attribute::

   <instance>
     <data xmlns="...">
      <user jr:template="">
         <region>BOSTON</region>
         <name />
      </user>
      <user>
         <region>NEW YORK</region>
         <name>Jasper</name>
      </user>
     </data>
   </instance>

Adding the ``jr:template`` attribute (value is irrelevant) to a repeatable node in the instance indicates that this repetition should be used as the default for all future repetitions that are created, and is not considered an actual repetition itself. In the above example, the form entry UI considers there to be **one** repetition already created (whose questions have default answers of ``NEW YORK`` and ``Jasper``, respectively). Any newly-created repetitions will have a default region of ``BOSTON``. The template is never included when submitting the instance.

If the only ``<user>`` in the instance above was the template user, then there would be **zero** users by default, and we would prompt whether we even want to create a user in the first place.

   In short, you pretty much always want to add a template declaration for your repeatable nodes.

A ``<repeat>`` is given the same visual hints in the UI as a ``<group>``. Again, the caption used for the banner header is taken from the ``<repeat>``'s ``<label>``. To associate a ``<label>`` with a ``<repeat>``, use the following syntax::

   <group>
     <label>[caption]</label>
     <repeat nodeset="...">
      ...
     </repeat>
   </group>

(this is standard xforms syntax, not a JavaRosa customization. the enclosing ``<group>`` must contain no other children)

It is possible to specify that a repeat may not have interactive additions or removals through the use of the attribute ``noAddRemove``. If this is set, the user will not be prompted to be able to add a new instance of a repeat in the user interface. This is helpful when use in conjunction with the next attribute, which allows you to specify a hint as to how many nodes should be available. Note that at the moment, the content of the attribute is unimportant (Any content in the attribute enables the property), but should probably be set as an xpath for now so that forms will be compatible in the future.

You can specify a hint in the ``count`` attribute for the form entry session as to how many instances of a repeat should be available to the user without any interaction necessary. This will pull an integer value from the data model, and ensure that at least that many instances are available to the user when they fill out the form. When used with the ``noAddRemove`` attribute this give you a way to automatically handle repeats without any interaction. This might look like::

   <group>
     <label>FOO</label>
     <repeat nodeset="..." jr:noAddRemove="true()" jr:count="/path/to/some/integer">
      ...
     </repeat>
   </group>

which would pull the integer inside of ``/path/to/some/integer`` and have the user create that many repetitions for FOO. If ``noAddRemove`` isn't set, after the repetitions are completed, the user will be asked if they want to add more.

You can limit how many repetitions can be created through a constraint on the repeatable node (e.g., ``<bind nodeset="[same as the nodeset of the <repeat>]" constraint="count(.) &lt;= 5" />``).

``<group>`` s and ``<repeat>`` s may be nested within each other arbitrarily deep.

Things we want to be able to do with ``<repeat>`` s but can't yet:
 1. Use dynamic logic to specify whether a repeat can be added to, rather than a static flag

Multi-lingual Support
===============================

XForms provides no way to dynamically support multi-lingual content in a single form. We've devised our own way where all language-dependent strings are replaced with 'text identifiers', which act as indexes into a multi-lingual dictionary in the model.

In the ``<model>``, you can add a multi-lingual dictionary with the following structure::

   <itext>
     <translation lang="[language name]">
      <text id="[text id]">
        <value>[this language's translation of the string corresponding to [text id]]</value>
      </text>
     </translation>
   </itext>

Additional ``<text>`` entries are added for each localizable string. The ``<translation>`` block is duplicated for each supported language. The content should be the same (same set of text ids) but with all strings translated to the new language. The language name in the ``lang`` attribute should be human-readable, as it is used to identify the language in the UI. A ``default=""`` attribute can be added to a ``<translation>`` to make it the default language, otherwise the first listed is used as the default.

Every place localized content is used (all ``<label>`` s and ``<hint>`` s) must use a converted notation to reference the dictionary:

``<label>How old are you?</label>`` is changed to ``<label ref="jr:itext('how-old')" />`` . With the corresponding entries in ``<itext>``::

   <translation lang="English">
     ...
     <text id="how-old">
      <value>How old are you?</value>
     </text>
     ...
   </translation>
   <translation lang="Spanish">
     ...
     <text id="how-old">
      <value>¿Cuantos años tienes?</value>
     </text>
     ...
   </translation>
   ...

Not every string must be localized. It is acceptable to intermix ``<label>`` s of both forms. Those which do not reference the dictionary will always show the same content, regardless of language.

In general, all text ids must be replicated across all languages. It is sometimes only a parser warning if you do not, but it will likely lead to headaches.

Even within a single language, it is helpful to have multiple 'forms' of the same string. For example, a verbose phrasing used as the caption when answering a question, but a short, terse phrasing when that question is shown in the form summary. We handle this as follows::

   <text id="how-old">
     <value form="long">How old are you?</value>
     <value form="short">Age</value>
   </text>

This is only supported for question captions (``<label>`` s inside user controls).

XForms Roadmap
===============================

The following features are on the roadmap to be added to the JavaRosa xforms engine:

 * support for more data types for ``<input>``, including date with time (``dateTime``), time (``time``), year (``gYear``), month (``gMonth``), year with month (``gYearMonth``), month with day (``gMonthDay``), email, etc.
 * support for ``<secret>``, ``<range>``, and ``<textarea>`` controls
 * full-featured and multi-lingual support for ``<output>``
 * full-featured and intuitive support for ``<upload>`` (data capture, like camera, audio recording, etc.)
 * native support for data types like GPS and RFID, where device capabilities permit
 * support for the ``appearance`` attribute to control look-and-feel
 * support for dynamic list selections through the ``<itemset>`` attribute
 * support for evaluating xpath predicates (immediate use case: helping support better dynamic list selections)
 * support for multiple instances and models (immediate use cases: provide reference data for dynamic list selections; provide a place to store interstitial variables you don't want submitted)
 * support for 'open' list selections, meaning you can type in an 'other' value not on the list
 * support for grouping questions into a single control (e.g., weight + unit (lbs/kgs))
 * allow binding controls to instance node attributes
 * support multi-lingual constraint messages
 * support multiple constraints on the same node (with unique messages)
 * support different severities of constraints, such as hard constraint (always rejected), soft constraint (warning only), and medium constraint (warning, but only accepted after some kind of override process (e.g., type your initials, enter your password...))
 * support ``calculate`` attribute 
 * support ``<setvalue>``, replace ``jr:preload``
 * support ``<hint>``, ``<help>``, and ``<alert>``
 * support pre-populatation (explicit or tied to another question) and add/delete restriction for ``<repeat>``

This does not imply any delivery date, however, or that any development resources have been allocated. This is simply a list of features we'd like to be able to support, some more strongly than others.
asd



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
