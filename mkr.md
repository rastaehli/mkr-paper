# Build Tool too Complex? Component Technology to the Rescue

# Abstract

This paper describes how a canonical model of building can be applied to simplify software construction.  We describe the model and provide a detailed example of its application in a software build tool which is both simpler and more powerful than the current state of the art.
The same model easily supports configuration, dependency injection, physical distribution and deployment.

The primary contributions of this work are the validation of the model in a practical tool, the tool itself as a prototype implementation.


# Intro

Building software from re-usable components is a vision that has guided many advances including Abstract Data Types [???], Object Oriented Programming [Kay definition of message passing], Component Architectures [Java Beans, EJB?, MS COM] and Service Oriented Architectures [SOA service discovery,composability].  But the realization of this vision has been frustrated by the reality of a complex and rapidly evolving software technology infrastructure.  While mechanical and electronics engineers can rely on well known scientific theory to measure and describe component behavior, a comparable theory of software components is still evolving and not well understood by practitioners.  The result is that component development and reuse for the most part is still limited to large scale components like libraries, applications and distributed services.

Is there any reason that we can't build with finer-grained re-usable software components?  Is there any reason we can't build everything from components?  This paper argues that we can; that we need a general component model that encompasses every ingredient in the process.  Such a model would enable build tools that are independent of any particular hardware or software technology.  We describe such a model, the QuA Component Model (QCM), and show how it can be applied in the implementation of a practical software build tool.  While still a prototype, we show this build tool is less complex and offers functionality surpassing that of a modern industrial build tool.  This provides evidence that the QCM leads to simpler and more powerful approaches to software construction.

Section [QCM] describes the component model, Section [Maven] gives an overview of how a modern build tool supports software builds, Section [MKR] describes implementation of a build tools based on the QCM, and Section [Analysis] compares the two approaches.  Finally, Section [Conclusion] summarizes the strengths of the QCM approach, the evidence supporting these claims, and questions to be answered by future work.

## QCM: A Canonical Model for Systems Composition
MKR is based on the Qua Component Model (QCM).  The goal for QCM is a canonical model for building things from components: a model complex enough to express every essential feature of other models, but with no unnecessary complexity. The model attempts to account for how computers are built from electronic components, how cake is made from flour, and yes, how software applications are built from source code.  

QCM is a work in progress that has yet to be validated in large scale applications.  The original goal of QCM was to support Quality-of-Service (QoS)-based dynamic adaptation, but the QoS aspects are beyond the scope of this paper.  Here we focus only on how the model can be used to advance the state-of-the-art for software build tools.

## Main Definitions:
In this paper, a component is a very general abstraction: it is anything that interacts with its environment through physical interfaces and discrete messages, and by physical interfaces we intend to include the invocation addresses for software objects executing on physical hardware.  This subsumes many similar concepts such as software objects and web services.

A meta-component is a description of a component:
- A specification with:
    - a type that identifies the (ideal) component behavior: what are its logical interfaces and how inputs (incoming messages) over time may determine outputs (outgoing messages).
    - a quality specification of tolerance for imprecision in output timing and values.
- A plan that describes how the component is built:
    - blueprint => literal instructions for building from dependencies.
    - dependencies => map role=>resource for all resources required for building.  A resource may be changed, consumed or become part of the built component.
    - configureable-attributes => map attribute=>value for all configurable attributes.
    - provisioner => agent to allocate dependency resources.
    - builder => agent to interpret and execute the blueprint.
    - manager => agent to control activation.
- An implementation reference that maps from logical interface names to the interface implementations.

We argue that this simple model is all that is needed to support reflection on how a complex software system is built.  It supports both description of built systems, and prescription of the requirements to be satisfied by an automated build tool.

## Component Lifecycle
The lifecycle of a component implementation is modeled as a sequence of states:
- Specified: the required behavior, location of interfaces, and quality are identified in a meta-component description.  Default location and quality may be inferred from the specification context.
- Planned: an implementation plan with explicit dependencies has been chosen.
- Provisioned: all leaves of the implementation tree identify component resources allocated to the build.
- Built: the implementation has been built, but is not yet allowed to interact.
- Configured: the implementation has configurable attributes set.
- Active: the component is allowed to interact with others.

These states describe the essential sequence of steps for creating a component.  We can create independent components in any order, but a given component must be _Specified_ before it is _Planned_, _Planned_ before it is _Provisioned_, and so forth.

Each state is realized by an agent that operates on the meta-component:

1. An _architect_ chooses the specification.
1. A _planner_ chooses the plan that can satisfy the specification.
1. A _provisioner_ allocates the resources required by the plan.
1. The plan's _builder_ builds the component.
1. The plan's _manager_ activates the component.

This defines a simple, but powerful, plugin architecture:  a generic _planner_ and _provisioner_ can build arbitrarily complex systems by finding a plan to satisfy a specification from QCM-compliant plan repositories, recursively building the plan's dependencies, then invoking the plan's builder to realize the component.  Primitive meta-components are those that have no dependencies and require no building.  Built meta-components are those with components that have already been built.  The provisioning process terminates when all dependencies are satisfied by primitive or built meta-components.

Figure <component-ontology> shows the definition of meta-component in the OWL (Web Ontology Language).
Using this ontology, we can create RDF representations for meta-components that can be stored and exchanged in messages.
In Section [How Does MKR Build Software] we show how repositories of these meta-components support distributed discovery of both primitive components and implementation plans to build complex components.

# What Does Maven Do and How?
Maven is currently the most popular [?] build tool for Java projects.  To avoid offending Maven evangelists I must quickly add that Maven can do more than build Java software.  Strictly speaking, it is a build framework that you may customize to build and deploy any information artifact; software or other.

In its simplest and most common use as a Java build tool, the Maven command "mvn -install" takes as input a directory hierarchy of Java source files plus a Project Object Model (POM) file and outputs a jar file.  The jar file is then "installed" in a maven "artifact" repository where it can be be referenced as a dependency by other projects.

As with QCM, we will describe the main terminology and build model defined by Maven.

## Maven Definitions:
- An _artifact_ is an immutable file identified by "coordinates" consisting of _group_, _artifact_, and _version_ identifiers.  For example the coordinates org.apache:axis:1.1 identify the immutable version 1.1 of the axis artifact (a ".jar" Java Archive file) from the Apache Software Foundation.
- A _Maven project_ is a collection of source files in a standard directory hierarchy with a POM file at the root.
- A _POM file_ describes the artifact to be built and explicitly declares all build dependencies on other artifacts.  The POM file may identify and configure any non-standard plugins used to build the project artifact.
- A _Maven repository_ supports storage and retrieval of artifacts by their coordinates.

Maven defines its standard build lifecycle as a sequence of build goals:
1. clean: remove any files derived from previous builds.
1. compile: derive Java classes (or other compiled files) from source.
1. test: execute automated tests and capture test results.
1. package: build the project artifact, such as a jar file.
1. install: store the artifact in a local maven repository.
1. deploy: push the artifact to a remote repository.

Maven's standardization of these concepts yields its greatest strength: the ability to reliably build a project even when the project files are moved to another organization and environment.  So long as a maven project build depends only on project files and immutable artifacts available from trusted repositories, the execution of the install goal will produce the same successful result in any maven environment.

Unlike a QCM plan that allows arbitrary dependencies,  Maven assumes a collection of project files and pre-built artifacts.   Each stage of the maven lifecycle refers to a transformation on this collection of files: compiled classes, test results, a package file, a repository artifact.  

For the purpose of this paper, we focus on how Maven builds software when executing the "install" goal.  

When invoked with any goal, Maven first reads and validates the POM file. From the POM, it learns the type and coordinates of the artifact to be produced.  When invoked with the install goal, Maven automatically executes compile, test, and package goals first to ensure the artifact is built and ready for installation.  

Because it is a plugin-framework, it also learns from the POM the artifacts that implement the plugin goals.  These plugins are downloaded, if necessary, from a trusted repository and executed to achieve each goal.  Many Java projects use the default plugins inherited from the standard Maven installation, but any project may be configured to use custom plugins.

The default compile plugin uses the configured Java Development Kit (JDK) and compile-time libraries identified in the POM file dependencies to compile all Java files in the folder src/java to class files written to (new directories in) target/classes.  The default test plugin compiles test sources from src/test/Java to target/test/classes, executes those tests and captures test results in target/test/output.  The default package plugin delegates to the appropriate packaging tool based on the POM file's declared artifact type.  For example, if the artifact type is "jar" then the JDK jar tool is invoked to copy everything in target/classes and all other files in src/resources into a Java archive library file in the target directory.  Finally, the default install plugin will copy the packaged artifact file to a local Maven repository with the specified coordinates. (The local Maven repository is a hierarchical directory structure storing each artifact in a path based on its coordinates.)

Maven coordinates support both a development "SNAPSHOT" of an artifact and an immutable released version.  During development, when a project may undergo rapid changes, the "SNAPSHOT" designation is used to indicate that an artifact is under development and may be overwritten at any time in the repository by a newer build.  A timestamp is appended to the actual artifact file name so that the name still effectively represents an immutable object.  However, when another project includes the SNAPSHOT designation on a dependency version it is explicitly declaring itself to be dependent on mutable artifact.  All such SNAPSHOT dependencies must be replaced with immutable versions before a project may used to release a new immutable artifact version.

# How Does MKR Do The Same?

MKR is a proof of concept implementation of a generic QCM builder.  Given the goal of building a component from a meta-component with only the type specified,  MKR searches its repositories for candidate meta-components that can satisfy the goal.  Each candidate meta-component either identifies an built component or contains an plan that can be executed to build the desired component.
If a candidate's plan has unresolved dependencies, these serve as subgoals for recursive invocations of MKR.

## Example
-   goal build running WebApp-1.0-* at http://www.myorg.domain:8080/myapp
-   plan: 
    - builder Deployer-x.x at http://www.myorg.domain:9000/deployer
    - dependency "warFile" -> build WebApp.war-1.0

For example, to deploy the latest revision of  a web application "WebApp-1.0-*" (version 1.0 that is still in development) to a server "www.myorg.domain", port 8080 with context "myapp", we define a MetaComponent (in Turtle RDF syntax):

    @prefix owl: http://owl.domain/owl
    @prefix qcm: http://qcm.domain/qcm
    @prefix my: mkr://myorg.domain/

    my:app owl:class qcm:MetaComponent;
	qcm:type my:type/WebApp;
	qcm:interface ("baseUrl" -> http://www.myorg.domain:8080/myapp );  // we know how we want people to access it
	qcm:builder my:builder/WebArchiveDeployer;  // we know the implementation will depend on a webArchive
	qcm:dependency ("webArchive" -> my:war/webapp:1.0-LATEST );  // and we know the name of the webArchive

In the example above, the webArchive attributes is specified with the "mkr:" protocol for the URI.
This URI has the form "mkr://<organization>/<resourceId>:<version>[-<update>]".
We specify "LATEST" update to rebuild the war file whenever any of its component are updated.

The details of syntax for this specification are described in more detail in Sidebar "MetaComponent".

Sidebar "MetaComponent":

    URIs with the "mkr:" protocol prefix have the form "//<organization>/<resourceId>[:<version>[-<revision>]]".

    If the optional version is omitted, the URI matches any version of the resource.

    If version is specified, but the optional revision is omitted, the URI identifies a final immutable version of a resource.

    If the revision "-LATEST" is specified, the URI identifies a resource that should be updated each time an implementation dependency is updated.

    The wildcard character "*" may be used in place of any component of the URI to match any value of the same component of a candidate URI.

    A qcm:type is a URI for a resource that names the behavior of the component.  
    
    A qcm:type resource, if it exists, should be an instance of owl:class qcm:Type and have attributes that describe the names of the component's interfaces, but introspection on the type is beyond the scope of this paper.
    
    A qcm:interface maps one of the type's interface names to the URI used to invoke that component interface.

    In the "webArchive" dependency we use the operator "->" as shorthand for an association from string to value: "webArchive" -> "src:my-webapp-1.0" translates to '_ owl:class qcm:association; qcm:key "web-archive"; qcm:value: src:my-webapp-1.0'.


Note that the implementation of the WebApp type is constrained by specifying the webArchive dependency and baseURL interface.  Since all attributes of a ComponentSpec represent restrictions on the implementation, specifying interfaces and implementation dependencies creates a parameterized type.  

We can then read this declaration into MKR and make it "active":

    MKR load: 'declaration.rdf'; activate: my:app.

MKR inspects the goal and finds it in the Specified state: the implementation plan is incomplete because it is missing a blueprint. 
MKR searches its known implementation repositories for candidate implementation plans that match the specification.
A candidate matches if all specified parts of the candidate conform to the the corresponding part of the specification.  

1) Search for components in local, then organizational, then global repositories in that order, terminating search when a full match is found
2) In each repository, search first by matching type, then restrict to implementations conforming to all dependency and interface constraints.
3) If a conforming candidate plan is found, a copy is made and specialized by binding dependencies and interfaces to constrained values.
3) If multiple component matches are found, return the most fully ready (already built) of the set.

Need to say something about where MKR runs, where implementation repositories reside, how ComponentSpecs get into the repositories and how they are updated.  Where builder resources are located and how they are invoked.

If a fully ready component is found, MKR binds the goal ComponentSpec implementation to this instance and returns the ComponentSpec in a ready state.

If a viable plan is found, its dependencies are readied by making a recursive call to MKR and updating the depedency ComponentSpec with it's implemented value.  Then the builder's "buildWith dependencies" operation is invoked with the ready dependency values.


================================= older text needs updating to turtle syntax approach and new example names ===================

This resource is another QCMComponentSpec that identifies the type of this resource and locates the project source files in an SVN repository:

A MKR object is an instance of the standard QCMBuilder type which, like Maven, is responsible for implementing QCMComponentSpec lifecycle transitions.  Also like Maven, it does this using an open-ended set of plugin build components.  The great simplicity of MKR comes from using the same QCMComponentSpec abstraction to represent both build goals and build implementation plans.  The ability to build complex systems comes from composing many simple implementation plans.

     =========
    As created above, the goal is not yet implemented, only typed and located.  In this state, the "ready" command is the same as the command sequence "plan provision build configure ready".
     ==============

The plan command searches the local implementation repository for plans with matching specs.  A spec matches if the specified values in the goal match: the type, dependency, and interface.  Wildcards are supported to allow an implementation plan to match a range of values.  In this example, the only match is the one our deployment engineer created for building an SVN project and deploying to a specified target:

    QCMContext 
	namespace: "http://myorg.domain/type" abbreviation: "my";
	namespace: "http://myorg.domain/script" abbreviation: "script".

    plan := QCMComponentSpec new
	type: "my:WebApp-1.0";
	dependency: "source" value:  "*";
	interface: "webAppBaseUrl" location: "*";
	implementation: (QCMArchitecture new
		builder: "http://build.myorg.domain/mkr-1.0";
		blueprint: "script:deployWebapp-1.0";
		dependency: "warFile" value: (
			QCMComponentSpec new
				type: "my:WarFile-1.0";
				dependency: "source" value: (plan typeDependencyAt: "source"));
		dependency: "target" value: (
			QCMComponentSpec new
				type: "my:WebDeploymentAccount";
				dependency: "baseUrl" value: (plan interfaceAt: "webAppBaseUrl")).

When matching this plan, MKR first finds that the wildcards allow a match, then it creates a copy of the plan and replaces the wildcard values with the goal values.  The result is a plan specialized for this goal.  The implementation portion of the plan may then use these specialized values during its execution.  In this example, the web application is to be implemented by a plugin builder that executes a deployment script


The matching algorithm binds strings from the goal type to the variables in the plan type as follows:

    appName = "my:DemoWebApp"
    version = "1.0"
    server = dev
    context = myapp
				
This is a backward chaining algorithm to discover and build the subcomponents required by each plan.  Figure "Planning the Webapp Deployment" shows a breadth-first enumeration of the resulting tree.  

			
Build Patterns	
		
- build a compiled java class from source file

    'java:class(package.ClassName)' depends on java compiler context: local file system, default sources and classes dirs, path from package name.  
	goal is class in local qua repository, as the java:class semantics does not imply the object is available to others.
	no need to build if class file result already exists with creation time more recent than source
- load a Java class into a JVM
- instantiate a Java class with specific constructor arguments
- build a target specific (configuration?) file from input source files and a standard tool like one that substitutes variables based on target context

    'namespace:goaltype(args...)'

    'cfg:targetConfigFile(

- from SVN project

    'svn:sourcesdir(projectName)' planner finds svn component that can create sources dir for project by checkout from SVN repository.
- build a jar library of classes from SVN project

    'java:jar(com.expd:my-project:1.0.0)' 
	could find in local qua repository (like local maven repository default directory)
- copy files from source directory to a target directory
- build a war file from SVN project
- install an immutable versioned build artifact (arbirary file) in a shared repository
- deploy a file to a remote target context (webapp deployment account for example) (replace components of the deployed component, prep for stop, disassemble, reassemble, start of new)
- stop, disassemble (and recycle old components), (recreate, reuse, and replace) and reassemble, then start a remote webapp.
- upgrade a ready component (provision replacement components, including new version of component libraries, then take offline, restart, reassembling with upgraded components, put online)
- upgrade a suite of components
- handle failure of upgrade with rollback of all changes in transaction.
- create a new version of a class, compile, upgrade test environment with new version, test, repeat till DEV tests pass, commit to source repository
- upgrade QA environment with new version, test, repeat creation of new version and deployment till QA tests pass. Tag this approved configuration of sources for release.
- release QA approved versions to archival repository, upgrade production components with new released version (in a transaction with rollback on error).



Since the type value is immutable, the type can be satisfied by a newly built instance or an old one; by a remote instance, a local copy, or even a copy from a different (trusted) organization.

# (Cost Benefit) Analysis

Remember that comparison of benefits is largely a wash because we've aimed to re-implement same functionality.  Difference will show only in what one or the other can't do easily.
Perhaps best to break it down by problem solved, times frequency of having to solve this problem in practice):
- repository component definition: cost of documentation, learning to use; benefit of being able to lookup immutable artifacts by name.
- identifying an immutable artifact: cost of writing coordinates in a POM file; benefit of being able to identify an immutable component.
- automated build: cost of specifying dependencies, resolving conflicts, acquiring components in local repository; benefits of correct build, speed of build.
Maven costs:
- 
Can do "build avoidance" by finding previously built components advertised in the local repository.
Can store reusable deployment configuration plans in source controlled repository for change management control, make these part of the definition of a deployment environment; a system-level component implementation.  This avoids the problems of trying to maintain knowledge of deployment configurations in the same source with the component being deployed: we don't want to have to consider the component changed every time we think of a new place to deploy it.

Figure [MavenWARExample] shows the POM file for building a simple Web application WAR file.  (A WAR file is a file that can be deployed into a web server container to implement a dynamic web site. )  Figure [MKRMyWebExample] shows the MKR plan for building the same WAR file from its dependencies.  Note how MKR separates responsibility for the WAR file build from other concerns, making this specification smaller and less complex than the Maven POM file.  MKR also 

[Maven] assumption about project files is somewhat at odds with its goal of being a universal build tool.  For example there seems to be no way to identify a local web service as a dependency to be composed with some other service.

# Conclusion

QCM is the first component model we are aware of that can describe all the components involved in building and deploying a modern distributed application.  This demonstrates the value of the model, but the model also enables a great simplification in design of tools such as MKR that promises a real advance in productivity: simpler tools are easier to maintain and to use.

# Future Work
Software components have long been promised as the solution to software development problems.  As in any other technology domain, it is generally faster to implement new systems from standard software components than to implement custom functionality from scratch.  Modern web applications, for example, are commonly build with off-the shelf frameworks, libraries and tools so that only the minimal specialized web pages and application logic need be written.

Ingredients versus interfaces.  Not just dependencies.  By specifying dependencies with MetaComponents, we are not restricted to declaring only behavior (logical interface) type.  We can also specify physical interface locations (distribution) and even parts (restrictions) of the implementation.  Could this be valuable in new language design?  If Java code could reference only QCM types, the runtime environment could perform dependency injection, automatically compute (jar-file) dependencies and other.

A component implementation does know its depednencies / ingredients: the parts it owns and may consume in its constructions.

## Composition of Extra-Functional Behavior

If we can construct an X-Service we can give this service HA-fault-tolerant, secured, rate-limited, highly-available, auditable and other extra-functional behavior with composable filters.  The plans for these filters include the service as a dependency and add additional resources and configuration to yeild better service quality.

HA-fault-tolerant: add monitors, failover and retry logic to reduce the chance of system errors returned to the client.
secured: add client authentication and permission checks to reduce the chance of unauthorized access to protected operations and information.
rate-limited: add rate monitoring and limit-exceeded responses to protect response times for clients that adhere to SLA limits.
auto-scaling: automatically add/remove duplicate and load-balanced instances of a stateless service to maintain targeted performance levels.
auditable: add usage logging to support auditing of who uses the service and how.

## Programming: object instantiation and initialization

Modern programming languages like Java, Javascript (ECMAScript 5) and Python provide 
- extra syntax for object construction and instantiation (public X(...), new X(params), new X(params), def __init__(self), X(params)), 
- more syntax to provide controlled access to configure object implementation properties
- still more libraries and design patterns to support dependency injection and factories.

Because the QCM is a canonical model of construction, it can be applied with a language to encapsulate this extra complexity behind a standard Meta-Object Protocol (MOP):
- if mkr is a singleton service that implements the MKR MOP
- mkr activate(desc)  // returns an active instance of component specified by desc (including config in desc)

Mkr handles object construction with default class constructor plan associated with the type in "desc".

Mkr handles configuration as part of the construction, taking configuration parameters from desc.plan.configuration.

Mkr supports dependency injection by configuring mkr with plans for each imported type.  For example, a test environment can configure mock implementation or test configurations of imported types with  regular implementations:
- mkr advertise(desc)  // during mkr setup, adds desc.plan for building plan.type to mkr repository
- mkr activate(desc.type)  // at any time later, returns an instance built from desc.plan

- mkr activate("MyType")  // short spec of desc with only type name (defined in mkr repository of imported types)
- mkr 

This simplfies programming by separating the concerns of initialization and configuration from program function.  Instead of a main program like this:
  import java.lang.Map;
  ...
  import javax.jdbc.DriverManager;

  class MyApp {
  ...
  public MyApp(Connection c, ...) {
    // initialize the instance of MyApp
  }

  private void doWork() {

  }

  public void main(String args[]) {
    String url = ... // read database URL configuration
    Map connectionProps = ... // read additional database configuration
    conn = DriverManager.getConnection(url, connectionProps);
    ...  // init all objects used in application initialization

    // ensure objects are initialized in correct sequence (before use)

    doWork();
  }
  }

We can write a generic main program like this:

  public void main(String args[]) {
    
    MKR mkr = new MKR();  // local object may connect to network of peers as configured in local deployment
    try {
        if (args[0] = "-f") {
            Desc desc = readDescFile(args[1]); // second arg is file with component description
            mkr.activate(desc);
        } else {
            mkr.activate(args[0]);  // first arg is type to build and activate
        }
    } catch(Exception e) {
        System.out.println("exception: "+e.message());
        showHelp();
    }
  }

Put all the imports and initialization dependencies in the component description file:

  {
    type: "MyApp",
    plan: {
        build: "new MyApp(conn, ...)",
        configure: {
            loops: "instance.setLoops(loops)",
            ...
        }
        dependencies: {
            conn: "TestEnvDBConn",
            ...
        }
        configureable-attributes: {
            loops: 200,
            ...
        }
  }

And simplify the class:

  import javax.jdbc.Connection;  // import only types needed in constructor and doWork

  class MyApp {
  ...
  public MyApp(Connection c, ...) {
    // initialize the instance of MyApp from constructor args
  }
  
  private void doWork() {
    // do application functionality
  }

  }
  