.. title: Infrastructure Standart Operating Procedures SOP
.. slug: infra-sop
.. date: 2015-04-23
.. taxonomy: Contributors/Infrastructure

======================
Infrastructure SOP SOP
======================

We use SOPs (standard operating procedures) to list how we run things in
Fedora. A better explanation is available in REDAME file.

A list of SOPs is available at
http://infrastructure.fedoraproject.org/infra/docs/

Use the same "Contact Information" section on all SOPs. Past that,
sections vary as needed.

Contents
========

1. Contact Information
2. SOP naming
3. Stuff every SOP should have
4. SOP formatting

Contact Information
===================

Owner
	 Fedora Infrastructure Team
Contact
	 #fedora-admin
Location
	 fedoraproject.org/wiki
Servers
	 app servers mediawiki runs on
Purpose
	 Keeps sysadmin-main sane

SOP naming
==========
SOP text file names should use lowercase with dashes.
Describe the service and end the page name with ".rst".

Stuff every SOP should have
===========================

Contact information
-------------------

Here's the template::

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
    <a brief description of the SOPs purpose

SOP formatting
==============

SOPs are written in ReStructuredText. To learn about the spec, read:

Quickstart
  http://docutils.sourceforge.net/docs/user/rst/quickstart.html
Quick references
  http://docutils.sourceforge.net/docs/user/rst/quickref.html
Full Specification
  http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html

The format is somewhat simple if you remember a few key points:

- Sections are deliniated by underlined texts.  The convention is:
  
  - Title has "=" above and below the title text, at least as many columns as the title itself.
  
  - Top level sections are underlined by "===" - at least as many columns as the section title in the line above.
  
  - Second level sections are underlined by "---"

  - Any of ``! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~`` are valid section deliniators.  
    If you need more than two section levels, choose between them but be sure to be consistent.

- Indents are significant.  Only indent for things like block quotes, nested lists etc. 
  Match the tabstop of the document you are editing.  Make note of the indentation level as 
  you nest lists.

- Use literal blocks for code and command sections. An indented section found after ``::`` and a newline will be processed as a literal block. Like this::
    
    Literal blocks can be nested into lists (think a numbered sequence of steps)

    1. Log into the thing
    
    2. run a command::

        this indented relative to the first content column of the list
        so it is a block quote

       This line begins at the first content column of the list,
       so it is considered a continuation of the list content.

    3. Log out of the thing.

- For inline literals (commands, filenames, anything that wouldn't make sense if translated, use your judgement) use double backticks, like this::

    You should specify your Fedora username and ssh key in ``~/.ssh/config`` to make connecting better.

- If nesting and mixing content types, use newlines liberally.  
  A bullet list doesn't need newlines, but if a list item's content spans more than one line, a newline *is* required.
  If a list is nested, the top level list should have newlines between list members.
    
