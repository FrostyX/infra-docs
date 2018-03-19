===================
 Purpose and scope
===================

This Service Level Expectations document is to help provide a high
level description of the services, equipment and duties of the Fedora
Infrastructure team. It will touch upon the responsibilities of both
customers and Infrastructure towards providing and receiving
services. 

Further documents will be added to cover each service in more
detail. This document is a work in progress and will be modified in
response to the ever changing goals and needs of the project.

================
 Group Overview
================

The consistency of the Fedora Infrastructure team is covered in the
external document: https://fedoraproject.org/wiki/Infrastructure 

The primary purpose of Fedora Infrastructure is to help produce
regular releases of the Fedora Operating System. Services which are
aligned with that purpose will be alloted a higher priority than other
services. 

Secondary purposes are to help provide services for other parts of the
Fedora Project to communicate effectively, gather information about
uses, and market their work.

========================
 Primary Business Hours
========================

Fedora Infrastructure's primary business hours are aligned with the
work schedule of Red Hat Inc. Normal hours should be seen as during
Monday through Friday 1400 UTC to 2300 UTC with US national holidays
and a 2 week end of year closure affecting staffing and response
times. 

Services outside of primary business hours are done on call and depend
on the availability of staff. 


============================
 Roles and Responsibilities
============================

Fedora Infrastructure to Community
----------------------------------
* To have staff present and available in appropriate IRC channels to
  answer questions during primary hours. 
* To have particular staff on-call during off hours so that community
  members can contact for Critical and Important services.
* Interact with community members with respect and courtesy.
* Work with community members to get accurate and thorough
  documentation of incidents, problems, or feature requests.
* When possible resolve the problem as soon as acknowledged.
* Communicate clearly estimated resolution times.
* Move items which can not be resolved within a reasonable time to
  future feature requests or close out.

Community Member to Fedora Infrastructure
-----------------------------------------
* Give full and detailed reports of the problem or requested service.
* Provide clear and complete contact information and times when
  available. 
* Leave alternative contacts who can also be available in case of
  vacation or other emergencies.
* When contacted by Fedora IT, respond back within 5 business days 

Fedora Infrastructure to Fedora Infrastructure
----------------------------------------------
* Have a clear schedule of reachable hours.
* Set and take regular vacation time to be rested.
* Rotate through days on-call in IRC and tickets.
* If adding a new service, be available outside of normal business
  hours to help debug problems.
* Follow procedures and checklists when adding or updating services.
* Help with regular audits of the documentation

==================================
 Definition of Service Priorities
==================================

The general design of service priorities is that of concentric circles
where items rely on services in their own circle or a circle below
them. 

1. Critical services are ones which Fedora Infrastructure will work on
   24x7 with 52 week coverage if an unplanned outage occurs. Services
   will be configured to be highly available with an estimated
   planned/unplanned uptime of 95%. Response time should be within 1
   hour.

2. Important services are ones which Fedora Infrastructure will work
   to be available 24x7 with 50 week coverage. Response times during
   primary hours should be 1 hour and outside of it should be 4
   hours. 

3. Normal services are ones which Fedora Infrastructure will work to
   be available during primary work hours. Problems outside of these
   hours will be looked at as people are available. The services may
   be available outside of these but are of a lower priority than
   important services.

4. Low priority services are ones which are not critical or important for the
   primary function of Fedora Infrastructure. They will be worked on
   and looked at during primary business hours. Response times should
   be within 1 business day. 

5. Third Party services are ones which Fedora Infrastructure has
   outsourced tools and services to. Uptimes, service hours, and
   coverage are dictated by the third party. Depending on the type of
   problem, Fedora Infrastructure will act as an intermediary or in
   the case of tools like retrace and COPR direct the user to talk
   with the service owners.

6. Deprecated services are ones which Fedora Infrastructure are no
   longer putting resources into. This may be because the project has
   completed its mission, the upstream software is dead, or the
   original reasons for the product are available. Problems with these
   services will be looked at during primary business hours. Responses
   may be mostly "Will Not Fix". 

========================
 Limitations on Support
========================
* Some services that are associated with Fedora are provided by third
  parties. Changes and outages which affect them are outside the
  control of Fedora Infrastructure.
* Fedora Infrastructure will prioritize issues and requests that
  affect multiple people or teams over a smaller group or individual.
* Fedora Infrastructure has limited budget and hours. Requests and
  features will be prioritized to fit within those. 
* Fedora Infrastructure is bound by the laws and regulations of the
  United States of America. This means that certain requests, changes
  and problems are outside the ability of members to deal with. 


==========
 Glossary
==========

Planned outage: A planned outage is one that is announced with enough
time to give most user's the ability to plan around it. 

Unplanned outage: An outage that occurs suddenly without proper
allowance for users to plan around it.

Scheduled outage: An outage which has been scheduled to occur but may
not have been announced with enough time for users to plan around it.

High Availability: Systems are available during specified operating
hours with any unplanned outages 'masked' by other tools. 

Continuous Operations: Systems are available 24 hours a day, 7 days a week
with no scheduled outages. Unplanned outages are possible during this
time.

Continuous Availability: Systems or applications are available 24x7
with no planned or unplanned outages. This is a combination of high
availability and continuous operations.

Level of availability:
| Percentage | Max outage time per day |
|     90%    |     144.0 minutes       |
|     95%    |      72.0 minutes       |
|     99%    |      14.4 minutes       |
|     99.9%  |       1.4 minutes       |

Commited Hours of Availability: Hours that an organization will have staff
available to help deal with issues with systems, services, and
applications. Also known as "Regular Business Hours"

Outage Hours: Total number of hours of outage considered normal for
calculating achieved availability. 

Response Time: The time between the users notification of the problem
and when the help desk will begin to work on that problem.

Resolution Update: The frequency of updates to tickets

Estimated Time of Resolution: 

Priority Levels:
 * Emergency: Problems which are site wide, and affect the core
     functions of the project.
 * Urgent: Problems which affect multiple functions and groups in the
     project. 
 * Normal: Problems which affect a single user from performing needed
     duties. 
 * Low: A request for service, instruction, information that has no
     immediate impact on services.


