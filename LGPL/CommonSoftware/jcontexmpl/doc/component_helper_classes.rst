========================
Component Helper Classes
========================

For each component class, a corresponding helper1 class is needed. In most cases this helper class will be used only by the framework, which means that the component developer will not have to look at it much. 

The framework uses the helper class to 

* instantiate the component implementation class
* get information about the component’s interface(s)
* get the CORBA POA Tie skeleton class for the component

A template for the helper class gets generated automatically by the build system if the Makefile contains the line::

    COMPONENT_HELPERS=on

The template has a file ending .java.tpl to ensure that no edited .java helper class gets overwritten. It is meant as a convenience for the component developer who will normally just have to remove the .tpl ending and then treat that Java file like his/her own, which requires checking it into CVS.

For our HelloDemo component, the helper class does not need to be changed (some Javadoc lines stripped from the listing):

.. highlightlang:: java

::

    23 package alma.demo.HelloDemoImpl;
    24 
    25 import java.util.logging.Logger;
    26 
    27 import alma.acs.component.ComponentLifecycle;
    28 import alma.acs.container.ComponentHelper;
    29 import alma.demo.HelloDemoOperations;
    30 import alma.demo.HelloDemoPOATie;
    31 import alma.demo.HelloDemoImpl.HelloDemoImpl;
    32 
    33 /**
    38  * To create an entry for your component in the Configuration Database, 
    39  * copy the line below into a new entry in the file $ACS_CDB/MACI/Components/Components.xml 
    40  * and modify the instance name of the component and the container: 
    42  * Name="HELLODEMO1" Code="alma.demo.HelloDemoImpl.HelloDemoComponentHelper" Type="IDL:alma/demo/HelloDemo:1.0" Container="frodoContainer"
    45  */
    46 public class HelloDemoComponentHelper extends ComponentHelper
    47 {
    52   public HelloDemoComponentHelper(Logger containerLogger)
    53   {
    54     super(containerLogger);
    55   }
    56 
    60   protected ComponentLifecycle _createComponentImpl()
    61   {
    62     return new HelloDemoImpl();
    63   }
    64 
    68   protected Class _getPOATieClass()
    69   {
    70     return HelloDemoPOATie.class;
    71   }
    72 
    76   protected Class _getOperationsInterface()
    77   {
    78     return HelloDemoOperations.class;
    79   }
    80 
    81 }

Only in special situations, when a Java component chooses to sometimes use Java binding classes, but other times send or receive serialized XML instead, then the helper class must be modified. This goes beyond the scope of this tutorial and will be described in a more specialized document.
An example for this can be found in alma.demo.XmlComponentImpl.XmlComponentComponentHelper from jcontexmpl/src/.