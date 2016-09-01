================================
Creating the HelloDemo Component
================================

---
IDL
---

A component’s functional interface must be specified as an interface in CORBA’s Interface Definition Language (IDL). An IDL file may contain more than one interface. The Java container framework does not require further restrictions[1]_ on the usage of the more exotic IDL constructs beyond what ACS already mandates for the definition of C++ components. 

.. [1] Currently there is a problem with IDL attributes which will be fixed later.

Using the ALMA directory structure, you should put IDL files in the <xxxx>/idl directory of your module. Our HelloDemo component is defined in jcontexmpl/idl/HelloDemo.idl.

Let’s look at the listing of HelloDemo.idl with line-by-line explanations. 


.. highlightlang:: idl

::

    01 #ifndef _HELLODEMO_IDL_
    02 #define _HELLODEMO_IDL_
    03 #include <acscomponent.idl>
    04 #pragma prefix "alma"
    05
    06 module demo
    07 {
    08     // a very simple component 
    09     interface HelloDemo : ACS::ACSComponent
    10     {
    11         string sayHello();
    12
    13         string sayHelloWithParameters(in string inString, 
                                            inout double inoutDouble, out long outInt);
    14     };
    15 };
    16 #endif

    
========  =============================================================
1, 2, 16  Standard include guard to avoid multiple inclusions of this IDL file
3         Mandatory include of acscomponent.idl (see line 9)
4         
          Standard CORBA directive that affects generated Java classes[2]_. The prefix pragma must come after the include statements (this restriction might be lifted in the future, but for now the TAO IFR would have a problem otherwise.)
          It defines the Java package that is prefixed to the package derived from the declared module (“demo”) and interfaces, resulting in classes like “alma.demo.HelloDemo”. 
          It is an existing ACS practice/restriction to only prefix one package level, e.g. not use “alma.archive” as a prefix. This might be changed in the future when some issues with this have been resolved. The prefix seemed to be transformed differently into Java packages and CORBA repository IDs.
6         
          Declaration of the CORBA module “demo”. A module may contain many interfaces.
          As a convention, there could be a 1:1 mapping between ALMA subsystems and modules, but this needs further discussions.
9         
          Interface declaration for the HelloDemo component. Every ALMA component’s IDL interface must inherit from ACS::ACSComponent. This ensures that all components have a unique instance name and a state visible to other components.
11, 13    Silly methods using CORBA types string, double, and long
========  =============================================================

.. [2] At least for what we care about here; primarily the prefix controls the CORBA repository ID.


After the IDL file HelloDemo.idl has been written, a couple of Java classes must be generated from it. 
The ALMA Makefile will take care of this, given the line (see jcontexmpl/src/Makefile)

::
    IDL_FILES = HelloDemo
    
The generated Java classes are then automatically compiled and put into /lib/HelloDemo.jar. They belong to the package alma.demo and can be categorized as

=======================  ======================================
Functional interface     HelloDemoOperations.class
Client-side CORBA        _HelloDemoStub.class, HelloDemo.class
Server-side CORBA        HelloDemoPOA.class, HelloDemoPOATie.class
Helper & holder classes  HelloDemoHelper.class, HelloDemoHolder.class
=======================  ======================================

Warning: don’t choose the same name for ``IDL_FILES`` as for (one of the values of) ``JARFILES`` in your Makefile!  Otherwise the jar files produced from the IDL and from your module’s code have the same name and one gets overwritten by the other. 

------------------------
Component Implementation
------------------------

The implementation of a Java component is a Java class that must implement
 * the functional interface (that corresponds to the IDL definition), and the 
 * ``alma.acs.component.ComponentLifecycle`` interface that the container uses to start and stop the component and to provide a callback handle.

The implementation class may inherit from any base class1; no such base class is required by the framework though, which saves the single inheritance option offered by Java. 

For components that don’t need inheritance for other purposes, there is a generic base class (``alma.acs.component .ComponentImplBase``) that provides standard implementations of the required methods from ComponentLifecycle and ACSComponent, just to keep the component code simpler.
 
In real life we would usually first create the component implementation class with just dummy implementations for all mandatory methods, e.g. using code generation features of an IDE like Eclipse, and then move on to the next steps described in sections 6 and below. This document will ignore the iterative process and look at the final implementations right away.

The functional interface for the HelloDemo component is HelloDemoOperations, a Java interface generated by the CORBA IDL compiler. 

By convention, we call the implementation class “HelloDemoImpl” and create it in the package “alma.demo.HelloDemoImpl”. It can be found under jcontexmpl/src in CVS.

.. highlightlang:: java


::

    01 /* legal info cut out
    19  */
    20 package alma.demo.HelloDemoImpl;
    21 import java.util.logging.Logger;
    22 import org.omg.CORBA.DoubleHolder;
    23 import org.omg.CORBA.IntHolder;
    24 import alma.ACS.ComponentStates;
    25 import alma.acs.component.ComponentLifecycle;
    26 import alma.acs.container.ContainerServices;
    27 import alma.demo.HelloDemoOperations;
    28 
    29 /** Javadoc cut out
    37  */
    38 public class HelloDemoImpl implements ComponentLifecycle, HelloDemoOperations
    39 {
    40   private ContainerServices m_containerServices;
    41   private Logger m_logger;
    42 
    43   /////////////////////////////////////////////////////////////
    44   // Implementation of ComponentLifecycle
    45   /////////////////////////////////////////////////////////////
    46   
    47   public void initialize(ContainerServices containerServices) {
    48     m_containerServices = containerServices;
    49     m_logger = m_containerServices.getLogger();
    50     m_logger.info("initialize() called...");
    51   }
    52   public void execute() {
    53     m_logger.info("execute() called...");
    54   }
    55   public void cleanUp() {
    56     m_logger.info("cleanUp() called..., nothing to clean up.");
    57   }
    58   public void aboutToAbort() {
    59     cleanUp();
    60     m_logger.info("managed to abort...");
    61   }
    62   
    63   /////////////////////////////////////////////////////////////
    64   // Implementation of ACSComponent
    65   /////////////////////////////////////////////////////////////
    66   
    67   public ComponentStates componentState() {
    68     return m_containerServices.getComponentStateManager().getCurrentState();
    69   }
    70   public String name() {
    71     return m_containerServices.getName();
    72   }
    73   
    74   /////////////////////////////////////////////////////////////
    75   // Implementation of HelloDemoOperations
    76   /////////////////////////////////////////////////////////////
    77   
    78   public String sayHello() {
    79     m_logger.info("sayHello called...");
    80     return "hello";
    81   }
    82   
    83   public String sayHelloWithParameters(String inString,
    84       DoubleHolder inoutDouble, IntHolder outInt) {
    85     m_logger.info("sayHello called with arguments inString=" + inString
    86         + "; inoutDouble=" + inoutDouble.value
    87         + ". Will return 'hello'...");
    88     outInt.value = (int) Math.round(Math.E * 10000000);
    89     return "hello";
    90   }
    91 }

======  ==========================
20      The package declaration in accord with the “subpackage-per-component-implementation” convention
21      Import for logging (standard JDK logger configured by ACS)
22-23   Imports for CORBA Holder classes for the OUT parameters in the sayHelloWithParameters method (as defined in [Ref9]_)
24      Import of the ComponentStates interface, needed to access (and optionally modify) the component state
25      Import for the mandatory Lifecycle interface
26      Import for the ContainerServices interface which the container provides to the component. From this interface, the component gets everything the container can do for it on request (that is, not transparently w/o the component noticing)
27      Import for the HelloDemoOperations interface, which defines the functional methods that a client of our component would use.
38      The component implementation class implements the functional and the lifecycle interface; it does not inherit from any base class (if so, there would be even less here to look at…)
40      Reference to the ContainerServices object that the container will give us (see line 47)
41      Ref to the logger object which we’ll get from the container (see line 49)
47-51   
        The initialize method (from ComponentLifecycle) is called by the container after creating an instance of our component. It provides us with the ContainerServices object. We don’t use it for much besides getting a logger and storing that reference in a separate variable for easy access.
        We could do some component initialization, and could throw a ComponentLifecycleException if this something had failed.
52-54   Implementation of the execute method (from ComponentLifecycle). This method will be called by the container after initialize() has returned. Only few components are foreseen to use both initialize and execute; here we just log a message.
        The component will not be available to clients before execute() has returned.
55-57
        Implementation of the cleanUp method (from ComponentLifecycle). This method will be called by the container when the component is getting dismissed by the ACS Manager, but only after the last call to any of the methods from HelloDemoOperations has returned. 
        This would be the place to free resources (other components etc)
58-61   Implementation of the aboutToAbort method (from ComponentLifecycle). This method may be called asynchronously when the container or the entire ALMA system must shut down without waiting for proper termination of all components. It is meant as a notification to the component to perform the most urgent actions before being forcefully removed at any time.
67-69   Implementation of the componentState() method (from ACSComponent).
        The component has it’s state managed by the container in a default way (but could also change it itself). Here we simply return the state which the container keeps for us; in real-world components, we could check the state of other components that ours depends upon, and then compute the state.
70-72   
        Implementation of the name() method (from ACSComponent). The name is the instance name of our component, e.g. something we can’t know at compile time. We get the name from the container, using the ContainerServices interface.
        For static components (instances known at deployment), this name is configured in the configuration database (CDB). For dynamic components, it’s either specified by the component which creates it, or a default name is chosen by the framework.
78-81   Implementation of the sayHello method which is declared in HelloDemoOperations. 
        Line 79 shows how to log a simple message.
83-90   Implementation of the sayHelloWithParameters method: we set the OUT parameter and leave the INOUT parameter untouched.
        Note the use of DoubleHolder and IntHolder for outgoing parameters, a concept that is not possible in Java without using these helper classes; they are therefore defined in the CORBA IDL-to-Java mapping spec. [Ref9]_
======  ==========================


Note that the implementation of the HelloDemo component is done with just one Java class. A real component would rather have one main implementation class that implements the component interfaces and uses other classes to perform the functional tasks.
