

Introduction
============

Purpose and Scope of the Document
---------------------------------

The purpose of this document is to give a practical introduction to writing Java components for ALMA software, using the ACS container/component framework. You should have read the section on Technical Architecture in [Ref1]_ or have learned otherwise about the concepts of container/component and XML binding classes to be used for ALMA software. You should also be familiar with the ALMA build environment (see [Ref8]_), even though a few things are explained redundantly in this tutorial.

While describing the steps involved in developing two sample components, I will try to provide information beyond the scope of the demo components in order to help using the framework for concrete ALMA subsystem development. However, this is not a concept or design document for the respective parts of ACS. It should help you to get started nonetheless.

The Java container/component model is fully integrated in ACS. It is meant to be used by ALMA subsystems or the parts of them that don’t have real-time requirements and don’t directly control hardware devices. 

Abbreviations
-------------

This document no longer maintains a separate section for abbreviations. Please use the ALMA software glossary [Ref13]_.

References
----------

.. [Ref1] ALMA Software Architecture, Section 5 (Technical Architecture) https://science.nrao.edu/facilities/alma/aboutALMA/Technology/ALMA_Computing_Memo_Series/draft/ALMASoftwareArchitecture.pdf
.. [Ref2] ACS Architecture http://www.eso.org/projects/alma/develop/acs/OnlineDocs/ACSArchitectureNL.pdf
.. [Ref3] ACS FAQ http://almasw.hq.eso.org/almasw/bin/view/ACS/AcsFAQ
.. [Ref4] Management and Access Control Interface Specification http://www.eso.org/projects/alma/develop/acs/OnlineDocs/Management_and_Access_Control_Interface_Specification.pdf
.. [Ref5] ACS Command Center User Guide http://www.eso.org/~almamgr/AlmaAcs/OnlineDocs/ACSCommandCenter/Acs_Command_Center_-_User_Guide.html 
.. [Ref6] ACS Error System http://www.eso.org/~almamgr/AlmaAcs/OnlineDocs/ACS_Error_System.pdf
.. [Ref7] CDB Tutorial http://www.eso.org/~almamgr/AlmaAcs/OnlineDocs/CDB.pdf
.. [Ref8] ALMA Software Development Tools and Integration Procedures, 10.13 http://www.eso.org/~almamgr/AlmaAcs/OnlineDocs/IntGuidelines.pdf 
.. [Ref9] OMG IDL to Java Language Mapping http://cgi.omg.org/cgi-bin/doc?formal/02-08-05
.. [Ref10] Eclipse download http://www.eclipse.org/downloads/
.. [Ref11] Eclipse ACS Plugin http://www.aoc.nrao.edu/~javarias/ACS/eclipse/
.. [Ref12] Castor XML Java binding framework http://castor.codehaus.org 
.. [Ref13] ALMA software glossary http://edm.alma.cl/forums/alma/dispatch.cgi/ipt70designreviews/showFile/100346/d20040616122447/Yes/COMP-70.15.00.00-003-A-GEN.pdf