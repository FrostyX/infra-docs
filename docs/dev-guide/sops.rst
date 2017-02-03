.. title: Developing Standard Operating Procedures

.. _develop-sops:

========================================
Developing Standard Operating Procedures
========================================

When a new application is deployed in Fedora, it is critical that you add a
standard operating procedure (SOP) for it. This documents how the application
is deployed in Fedora. Consult the current :ref:`sops` and if one is missing,
please add it.

You can modify this documentation or any of the current :ref:`sops` by making
a `pull request <https://docs.pagure.org/pagure/usage/pull_requests.html>`_ to
the `Pagure project <https://pagure.io/infra-docs/>`_.


Adding a Standard Operating Procedure
=====================================
To add a standard operating procedure, create a new `reStructedText
<http://www.sphinx-doc.org/en/stable/rest.html>`_ file in the `sop
directory <https://pagure.io/infra-docs/blob/master/f/docs/sops>`_
and then add it to the `index file
<https://pagure.io/infra-docs/blob/master/f/docs/sops/index.rst>`_.

SOP text file names should use lowercase with dashes. Describe the service and
end the page name with ".rst".


Stuff every SOP should have
===========================

Here's the template for adding a new SOP::

    =========
    SOP Title
    =========
    Provide a brief description of the SOP here.

    Contact Information
    ===================
    Owner
      <usually, Fedora Infrastructure Team>
    Contact
      <stakeholder fas groups, individuals, IRC channels to find the action>
    Location
      <Relevant URIs, etc>
    Servers
      <affected machines>
    Purpose
      <a brief description of the SOPs purpose>

    Sections Describing Things
    ==========================
    Put detailed information in these sections

    A Helpful Sub-section
    ---------------------
    You can even have sub-sections.

    A Sub-section of a sub-section
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If a current SOP does not follow this general template, it should be updated
to do so.


SOP Formatting
==============
SOPs are written in ReStructuredText. To learn about the spec, read:

* `Quickstart <http://docutils.sourceforge.net/docs/user/rst/quickstart.html>`_

* `Quick references <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_

* `Full Specification <http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html>`_

* `Sphinx reStructuredText <http://www.sphinx-doc.org/en/stable/rest.html>`_

The format is somewhat simple if you remember a few key points:

- Sections are deliniated by underlined texts.  The convention is:

  - Title has "=" above and below the title text, at least as many columns as the title itself.

  - Top level sections are underlined by "===" - at least as many columns as the section title in
    the line above.

  - Second level sections are underlined by "---"

  - Any of ``! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~`` are valid section
    deliniators. If you need more than two section levels, choose between them but be sure to be
    consistent.

- Indents are significant.  Only indent for things like block quotes, nested lists etc.
  Match the tabstop of the document you are editing.  Make note of the indentation level as
  you nest lists.

- Use literal blocks for code and command sections. An indented section found after ``::`` 
  and a newline will be processed as a literal block. Like this::

    Literal blocks can be nested into lists (think a numbered sequence of steps)

    1. Log into the thing

    2. run a command::

        this indented relative to the first content column of the list
        so it is a block quote

       This line begins at the first content column of the list,
       so it is considered a continuation of the list content.

    3. Log out of the thing.

- For inline literals (commands, filenames, anything that wouldn't make sense if translated, use
  your judgement) use double backticks, like this::

    You should specify your Fedora username and ssh key in ``~/.ssh/config`` to make connecting
    better.

- If nesting and mixing content types, use newlines liberally.
  A bullet list doesn't need newlines, but if a list item's content spans more than one line,
  a newline *is* required. If a list is nested, the top level list should have newlines between
  list members.
