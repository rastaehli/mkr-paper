# Build Tool too Complex? Component Technology to the Rescue

# Abstract

This paper describes how a canonical model of building can be applied to simplify software construction.  We describe such a canonical model based on previous research into software component technology then provide a detailed example of its application in a software build tool.  

We show how this enables both a simpler and more powerful tool than the most popular build tool currently in use.
Because the model is not specific to software construction it easily supports configuration, dependency injection, physical distribution and deployment.

The primary contributions of this work are the validation of the model in a practical tool, the tool itself as a prototype implementation.


# Intro

Building software from re-usable components is a vision that has guided many advances including Abstract Data Types [???], Object Oriented Programming [Kay definition of message passing], Component Architectures [Java Beans, EJB?, MS COM] and Service Oriented Architectures [SOA service discovery,composability].  But the realization of this vision has been frustrated by the reality of a complex and rapidly evolving software technology infrastructure.  While mechanical and electronics engineers can rely on well known scientific theory to measure and describe component behavior, a comparable theory of software components is still evolving and not well understood by practitioners.  The result is that component development and reuse for the most part is still limited to large scale components like libraries, applications and distributed services.

Is there any reason that we can't build with finer-grained re-usable software components?  Is there any reason we can't build everything from components?  This paper argues that we can; that we need a general component model that encompasses every ingredient in the process.  Such a model would enable build tools that are independent of any particular hardware or software technology.  We describe such a model, the QuA Component Model (QCM), and show how it can be applied in the implementation of a practical software build tool.  While still a prototype, we show this build tool is less complex and offers functionality surpassing that of a modern industrial build tool.  This provides evidence that the QCM leads to simpler and more powerful approaches to software construction.

Section [QCM] describes the component model, Section [Maven] gives an overview of how a modern build tool supports software builds, Section [MKR] describes implementation of a build tools based on the QCM, and Section [Analysis] compares the two approaches.  Finally, Section [Conclusion] summarizes the strengths of the QCM approach, the evidence supporting these claims, and questions to be answered by future work.

## QCM: A Canonical Model for Systems Composition
MKR is based on the Qua Component Model (QCM).  The goal for QCM is a canonical model for building things from components: a model complex enough to express every essential feature of other models, but with no unnecessary complexity. The model attempts to account for how computers are built from electronic components, how cake is made from flour, and yes, how software applications are built from source code.  

QCM is a work in progress that has yet to be validated in large scale applications.  The original goal of QCM was to support QoS-based dynamic adaptation, but the QoS aspects are beyond the scope of this paper.  Here we focus only on how the model can be used to advance the state-of-the-art for software build tools.

## Main Definitions:
A component is a very general abstraction: it is anything that interacts with its environment through physical interfaces and discrete messages, and by physical interfaces we intend to include the invocation addresses for software objects executing on physical hardware.  This subsumes many similar concepts such as software objects and web services.

A meta-component is a description of a component.
- The meta-component has a specification with:
   -- a type that identifies the (ideal) component behavior: what are its logical interfaces and how inputs (incoming messages) over time may determine outputs (outgoing messages).
   -- a quality specification of tolerance for imprecision in output timing and values.
- It has an interface map from logical interfaces to a URI for the actual interface.
- It has an implementation plan that describes how the component is built:
   -- a builder that implements build and disassemble operations.
   -- a blueprint that provides assembly instructions for the builder.
   -- a dependency map for blueprint resource requirements.

The implementation plan's builder, blueprint, and dependency values are themselves meta-components that provide access to component interfaces (or at least a way to build them).  
A primitive meta-component describes an opaque resource or literal value: it has valid interface URIs (or literal values), but no implementation plan. 
The dependencies of an implementation plan define branches of an implementation tree that, when completed, will have all primitive meta-components at its leaves. 

Figure <component-ontology> shows the definition of meta-component in the OWL (Web Ontology Language).
Using this ontology, we can create RDF representations for components that can be stored and exchanged in messages.
In Section <How Does MKR Do The Same> we show how repositories of these meta-components support distributed discovery of both primitive components and implementation plans to build complex components.

QCM meta-components support reflection to both inspect and manipulate the implementation state of a component. 
The lifecycle of a component implementation is modeled as a sequence of states:
- Specified: the required behavior, location of interfaces, and quality are identified in a meta-component description.  Default location and quality may be inferred from the specification context.
- Planned: an implementation plan with explicit dependencies has been chosen.
- Provisioned: all leaves of the implementation tree are primitive; they identify component resources allocated to the build.
- Assembled: the implementation has been built, but is not yet allowed to interact.
- Active: the component is allowed to interact with others.

The order of these states is significant: there is no point to planning an implementation until we know what is required; the component cannot be fully provisioned until planned, and so forth.  
#Of course, the leaves of an implementation tree are always provisioned with ready components.  
#Consider the case where you want to interact with an online shopping cart on your home PC:  The distributed abstraction of the shopping cart component to record your purchases may be constructed from the already ready components including the remote shopping service, and local and remote networking components.

QCM models all state transitions as the execution of a plan consisting of a blueprint, an agent to interpret it, and a collection identifying any subcomponents required.  The allows a different agent to be used for each transition.  In particular, the plan to assemble a component from subcomponents consists of a blueprint, a builder agent to interpret the blueprint, and the subcomponents required for assembly.  This enables a simple, but powerful, plugin architecture:  so long as plugin builder agents conform to the QCM model, a generic build tool can construct arbitrarily complex systems by delegating specialized tasks to the builder agents. 

Figure "QCM Implementation Plan": visualize entities and relations: the plan has a map from dependency names to meta-components required for assembly, the blueprint specifies how to assemble them, and the builder agent performs the assembly according to the blueprint.  

1. assembly and disassembly, for locating and provisioning, and so forth.
1. A repository discovery plan has a strategy for finding repositories to search for components for a given interface location.
1. An implementation search plan has a strategy for finding and selecting an implementation for a type, repositories to search for implementations and a search agent to interpret the strategy.
1. A provisioning plan has a strategy for allocating subcomponent implementations, repositories to search for implementations, and a provisioner agent to interpret the strategy.
1. An assembly plan has a blueprint for assembly, subcomponents required, and a builder agent to interpret the blueprint. (assemble)
1. A startup plan has a blueprint for startup actions, subcomponents to be started, and a startup agent to interpret the blueprint.
1. A shutdown plan has a blueprint for shutdown actions, subcomponents to be shutdown, and a shutdown agent to interpret the blueprint. 
1. A disassembly plan has a blueprint for disassembly, subcomponents to be recycled, and a recycling agent to interpret the blueprint.
1. A recycling plan has a blueprint for release of , subcomponents to be recycled, and a recycling agent to interpret the blueprint.

Of course software build tools are also primarily concerned with building from dependencies, but sometimes this is lost amid specialized concerns of building compiled objects from source, building packages and so forth.

What Does Maven Do and How?
Maven is currently the most popular [?] build tool for Java projects.  To avoid offending Maven evangelists I must quickly add that Maven can do more than build Java software.  Strictly speaking, it is a build framework that you may customize to build and deploy any information artifact; software or other.

In its simplest and most common use as a Java build tool, the Maven command "mvn -install" takes a directory hierarchy of Java and related project source files, plus a Project Object Model (POM) file, as input and builds and deploys the default output artifact; a jar file for example.

As with QCM, we will describe the main terminology and build model defined by Maven.

Main Definitions:
- An artifact is an immutable file identified by "coordinates" consisting of group, artifact, and version identifiers.  For example the coordinates org.apache:axis:1.1 identify the immutable version 1.1 of the axis artifact (a ".jar" Java Archive file) from the Apache Software Foundation.
- A Maven project is a collection of source files in a standard directory hierarchy with a POM file at the root.
- A POM file describes the artifact to be built and explicitly declares all build dependencies on other artifacts.  The POM file may identify and configure any non-standard plugins used to build the project artifact.
- A Maven repository supports storage and retrieval of artifacts by their coordinates.

Maven defines its standard build lifecycle as a sequence of build goals :
1. clean: remove any files derived from previous builds.
1. compile: derive Java classes (or other compiled files) from source.
1. test: execute automated tests and capture test results.
1. package: build the project artifact, such as a jar file.
1. install: store the artifact in a local maven repository.
1. deploy: push the artifact into a shared repository.

Maven's standardization of these concepts yields its greatest strength: the ability to reliably build a project even when the project files are moved to another organization and environment.  So long as a maven project build depends only on project files and immutable artifacts available from public repositories, the execution of the install goal will produce the same successful result in any maven environment.

Unlike the QCM lifecycle that describes generic stages in building a component, the Maven lifecycle assumes an artifact is build from a collection of project files.   Each stage of the maven lifecycle refers to a transformation on this collection of files: compiled classes, test results, a package file, a repository artifact.  This assumption about project files is somewhat at odds with its goal of being a universal build tool.  The fact that you can create new plugins for any build goal does not change this fact of the Maven build lifecycle.

For the purpose of this paper, we deliberately ignore many features of Maven that are not relevant to building software.  The prime feature that is relevant is how Maven executes the "install" goal.  

When invoked with any goal, Maven first reads and validates the POM file.  From the POM, it learns the type and coordinates of the artifact to be produced.  When invoked with the install goal, Maven automatically executes compile, test, and package goals first to ensure the artifact is built and ready for installation.  

Because it is a plugin-framework, it also learns from the POM the artifacts that implement the plugin goals.  These plugins are downloaded, if necessary, from a trusted repository and executed to achieve each goal.  Many Java projects use the default plugins inherited from the standard Maven installation, but any project may be configured to use custom plugins.

The default compile plugin uses the configured Java Development Kit and compile-time libraries identified in the POM file dependencies to compile all Java files in the folder src/java to class files written to (new directories in) target/classes.  The default test plugin compiles test sources from src/test/Java to target/test/classes, executes those tests and captures test results in target/test/output.  The default package plugin delegates to the appropriate packaging tool based on the POM file's declared artifact type.  For example, if the artifact type is "jar" then the JDK jar tool is invoked to copy everything in target/classes and all other files in src/resources into a Java archive library file in the target directory.  Finally, the default install plugin will copy the packaged artifact file to a Maven repository with the specified coordinates.

Maven coordinates support both a development "SNAPSHOT" of an artifact and an immutable released version.  During development, when a project may undergo rapid changes, the "SNAPSHOT" designation is used to indicate that an artifact is under development and may be overwritten at any time in the repository by a newer build.  A timestamp is appended to the actual artifact file name so that the name still effectively represents an immutable object.  However, when another project includes the SNAPSHOT designation on a dependency version it is explicitly declaring itself to be dependent on mutable artifact.  All such SNAPSHOT dependencies must be replaced with immutable versions before a project may used to release a new immutable artifact version.

# How Does MKR Do The Same?

The QCM model supports building through backward chaining:   MKR starts with the specification of a goal and searches its repositories for candidate meta-components that match the goal.  Each candidate either identifies an active component or contains an implementation plan that can be executed to build a active component.
If a candidate's implementation plan has unresolved dependencies, these serve as subgoals for a recursive invocation of MKR.

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

    The actual resource should be an instance of owl:class qcm:Type and have attributes that describe the names of the component's interfaces, 
    but introspection on the type is beyond the scope of this paper.
    
    A qcm:interface maps one of the type's interface names to the URI used to invoke that component interface.

    In the "webArchive" dependency, "x -> y" is shorthand for a an association from string to value: 

    "webArchive" -> "src:my-webapp-1.0" translates to '_ owl:class qcm:association; qcm:key "web-archive"; qcm:value: src:my-webapp-1.0'.


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
    As created above, the goal is not yet implemented, only typed and located.  In this state, the "ready" command is the same as the command sequence "plan provision assemble ready".
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
- deploy a file to a remote target context (webapp deployment account for example) (replace components of the deployed component, prep for stop, dis-assemble, re-assemble, start of new)
- stop, disassemble (and recycle old components), (recreate, reuse, and replace) and re-assemble, then start a remote webapp.
- upgrade a ready component (provision replacement components, including new version of component libraries, then take offline, restart, re-assembling with upgraded components, put online)
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

Figure [MavenWARExample] shows the POM file for building a simple Web application WAR file.  (A WAR file is a file that can be deployed into a web server container to implement a dynamic web site. )  Figure [MKRMyWebExample] shows the MKR plan for assembling the same WAR file from its dependencies.  Note how MKR separates responsibility for WAR file assembly from other concerns, making this specification smaller and less complex than the Maven POM file.  MKR also 

# Conclusion

QCM is the first component model we are aware of that can describe all the components involved in building and deploying a modern distributed application.  This demonstrates the value of the model, but the model also enables a great simplification in design of tools such as MKR that promises a real advance in productivity: simpler tools are easier to maintain and to use.

# Future Work
Software components have long been promised as the solution to software development problems.  As in any other technology domain, it is generally faster to implement new systems from standard software components than to implement custom functionality from scratch.  Modern web applications, for example, are commonly build with off-the shelf frameworks, libraries and tools so that only the minimal specialized web pages and application logic need be written.

Ingredients versus interfaces.  Not just dependencies.  By specifying dependencies with MetaComponents, we are not restricted to declaring only behavior (logical interface) type.  We can also specify physical interface locations (distribution) and even parts (restrictions) of the implementation.  Could this be valuable in new language design?  If Java code could reference only QCM types, the runtime environment could perform dependency injection, automatically compute (jar-file) dependencies and other.

A component implementation does know its depednencies / ingredients: the parts it owns and may consume in its constructions.

