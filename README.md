# hMAS-Java

A library for handling resources in Hypermedia Multi-Agent Systems based on the HyperAgents ontologies:
- [Hypermedia MAS Core Ontology](https://purl.org/hmas/core) (see [core](https://github.com/danaivach/hmas-core))
- [Hypermedia MAS Interaction Ontology](https://purl.org/hmas/interaction) (see [interaction](https://github.com/danaivach/hmas-interaction))

The library can be used to execute actions based on a given signifier. It allows for the registration of custom payload and protocol bindings, and provides built-in bindings for the following protocols:
- HTTP (see [bindings](https://github.com/danaivach/hmas-bindings))

### Table of Contents
- [Getting Started](#getting-started)
- [Retrieving and Parsing Resource Profiles](#retrieving-and-parsing-resource-profiles)
- [Creating and Writing Resource Profiles](#creating-and-writing-resource-profiles)
  - [Agent Profiles](#agent-profiles)
  - [Artifact Profiles](#artifact-profiles)
  - [Workspace Profiles](#workspace-profiles)
  - [Hypermedia MAS Platform Profiles](#hypermedia-mas-platform-profiles)
- [Interacting Through Signifiers](#interacting-through-signifiers)
  - [Executing Actions](#executing-actions)
  - [Registering Custom Payload and Protocol Bidnings](#creating-custom-payload-and-protocol-bindings)
- [Credits and Thanks](#credits-and-thanks)
  - [Citation](#citation)
  - [Acknowledgements](#acknowledgements)

## Getting Started
You can add hMAS-Java to your project with [JitPack](https://jitpack.io/) to directly use the following libraries:
- [hMAS-core](https://github.com/danaivach/hmas-core)
- [hMAS-interaction](https://github.com/danaivach/hmas-interaction)
- [hMAS-bindings](https://github.com/danaivach/hmas-bindings)

### Add the JitPack repository to your build file

Gradle:

```groovy
allprojects {
  repositories {
    ...
    maven { url 'https://jitpack.io' }
  }
}
```

Maven:
```
<repositories>
  <repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
  </repository>
</repositories>
```

### Add a dependency to WoT-TD-Java

Gradle:
```groovy
implementation 'com.github.danaivach:hmas-java:main-SNAPSHOT'
```

Maven:
```
<dependency>
  <groupId>com.github.danaivach</groupId>
  <artifactId>hmas-java</artifactId>
  <version>main-SNAPSHOT</version>
</dependency>
```

## Retrieving and Parsing Resource Profiles
To retrieve and parse a _resource profile_ from a URL:
```java
ResourceProfile profile = ResourceProfileGraphReader.readFromURL(TDFormat.RDF_TURTLE, url);
```
Or from a local file:
```java
ResourceProfile profile = ResourceProfileGraphReader.readFromFile(TDFormat.RDF_TURTLE, filePath);
```
Or just parse it from a string:
```java
ResourceProfile profile = ResourceProfileGraphReader.readFromString(TDFormat.RDF_TURTLE, myProfile);
```
To retrieve and parse a resource profile of an _artifact_, you can directly use the artifact profile reader, e.g.:
```java
ResourceProfile profile = ResourceProfileGraphReader.readFromString(TDFormat.RDF_TURTLE, artifactProfile);
```
To retrieve and parse a resource profile of an _agent_, you can directly use the agent profile reader, e.g.:
```java
AgentProfile profile = AgentProfileGraphReader.readFromString(TDFormat.RDF_TURTLE, agentProfile);
```

## Creating and Writing Resource Profiles
### Agent profiles:

To create a resource profile of an _agent_:
```java
Agent agent = new Agent.Builder()
        .setIRIAsString("https://example.org/profiles/1#agent")
        .addSemanticType("https://example.org/vocabularies/bdi#BDIAgent")
        .build()

ResourceProfile profile = new ResourceProfile.Builder(agent)
        .setIRIAsString("https://example.org/profiles/1")
        .build();
```
The above code snippet creates a `ResourceProfile` of a BDI `Agent`. The locations of the resource profile and the agent are (optionally) provided. To serialize the resource profile in Turtle:
```java
String agentProfile = new ResourceProfileGraphWriter(profile)
        .setNamespace("bdi", "https://example.org/vocabularies/bdi#")
        .write();
```
The generated resource profile:
```turtle
@base <https://example.org/profile> .
@prefix hmas: <https://purl.org/hmas/> .
@prefix bdi: <https://example.org/vocabularies/bdi#> .

<> a hmas:ResourceProfile;
  hmas:isProfileOf <https://example.org/profiles/1#agent> .

<https://example.org/profiles/1#agent> a hmas:Agent, bdi:BDIAgent.
```
### Artifact Profiles 
To create a resource profile of an _artifact_:
```java
Artifact artifact = new Artifact.Builder()
        .addSemanticType("https://saref.etsi.org/core/LightSwitch")
        .setIRIAsString("https://example.org/profiles/2#artifact")
        .build();

ResourceProfile profile =
            new ResourceProfile.Builder(artifact)
                    .setIRIAsString("https://example.org/profiles/2")
                    .exposeSignifier(togglable);
                    .build();
```
The above code snippet creates an `ResourceProfile` of an `saref:LightSwitch` `Artifact` (see the [SAREF Ontology](https://saref.etsi.org/core/v3.1.1/)https://saref.etsi.org/core/v3.1.1/). The profile exposes a `Signifier` for indicating that the light switch is `togglable`. This Signifier can be defined in the following manner:
```java
Signifier signifier = new Signifier.Builder(new ActionSpecification.Builder(toggleForm)
                            .addSemanticType("https://saref.etsi.org/core/ToggleCommand")
                            .setInputSpecification(inputSpecification)
                            .build())
                    .build()   
```
This signifier signifies an `ActionSpecification` of an action that has the semantic type `saref:ToggleCommand`. The action requires an input defined in an `IOSpecification`, and is implemented through a `toggleForm` `Form`, which is a type of hypermedia control.

An input or output can be defined in an `IOSpecification` that defines contraints on the expected input or, respectively output, in the form of a [SHACL shape](https://www.w3.org/TR/shacl/#shapes). An input or output specification can be of type `StringSpecification`, `IntegerSpecification`, `DoubleSpecification`, `FloatSpecification`, `BooleanSpecification`, `ValueSpecification` (in case expected values are identified by IRIs), or `QualifiedValueSpecification` (in case expected values have properties). For example, to specify that the input should be a `saref:State` that `saref:hasValue` an `int` value: 
```java

QualifiedValueSpecification inputSpecification = new QualifiedValueSpecification.Builder()
            .addRequiredSemanticType("https://saref.etsi.org/core/State")
            .setRequired(false)
            .addPropertySpecification("https://saref.etsi.org/core/hasValue"),
                    new IntegerSpecification.Builder()
                            .setName("Toggle Value")
                            .setDescription("Toggle the light switch to the selected value")
                            .setDefaultValue(0)
                            .setRequired(true)
                            .build())
            .build();
```
Implementation details of an action can be defined in a `Form` as an [HCTL Form](https://www.w3.org/2019/wot/hypermedia#Form). For example, to specify that a request with the PUT HTTP method should be sent to `http:switch.example.org/state`:
```java
Form toggleForm = new Form.Builder("http:switch.example.org/state")
        .setMethodName("PUT")
        .setContentType("application/light+json")
        .setIRIAsString("https://example.org/profiles/2#form")
        .build();
```
To serialize the artifact profile in Turtle:
```java
String artifactProfile = new ResourceProfileGraphWriter(profile)
        .setNamespace("saref", "https://saref.etsi.org/core/")
        .write();
```
The generated artifact profile:
```turtle
@base <https://example.org/profiles/2> .
@prefix hctl: <https://www.w3.org/2019/wot/hypermedia#> .
@prefix htv: <http://www.w3.org/2011/http#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xs: <http://www.w3.org/2001/XMLSchema#> .
@prefix saref: <https://saref.etsi.org/core/> .
@prefix hmas: <https://purl.org/hmas/> .

<> a hmas:ResourceProfile;
  hmas:isProfileOf <#artifact> ;
  hmas:exposesSignifier [ a hmas:Signifier ;
    hmas:signifies [  a sh:NodeShape;
     sh:class hmas:ActionExecution, saref:ToggleCommand;
     sh:property [
       sh:path prov:used;
       sh:minCount "1"^^xs:int;
       sh:maxCount "1"^^xs:int;
       sh:hasValue <#form>
     ], [
      sh:path hmas:hasInput ;
      sh:qualifiedValueShape [ a sh:Shape;
        sh:class saref:State;
        sh:property [
          sh:path saref:hasValue;
          sh:name "Toggle Value";
          sh:description "Toggle the light switch to the selected value";
          sh:defaultValue "0"^^xs:int;
          sh:minCount "1"^^xs:int;
          sh:maxCount "1"^^xs:int;
          sh:datatype xs:int
        ] 
      ] ;
      sh:qualifiedMaxCount "1"^^xs:int; 
    ] 
   ]
  ].

 <#artifact> a hmas:Artifact, saref:LightSwitch .
  
<#form> a hctl:Form;
  hctl:hasTarget <http:switch.example.org/state> ;
  htv:methodName "PUT" ;
  hctl:forContentType "application/light+json" .
```
### Workspace Profiles
To create a resource profile of a _workspace_:
```java
Workspace workspace = new Workspace.Builder()
        .addContainedResource(agent)
        .addContainedResource(artifact)
        .setIRIAsString("https://example.org/profiles/3#workspace")
        .build();

ResourceProfile profile = new ResourceProfile.Builder(workspace)
        .setIRIAsString("https://example.org/profiles/3")
        .build() ;
```
The above code snippet creates a `ResourceProfile` of a `Workspace` that contains the agent and the artifact previously defined. The resource profile can be serialized in Turtle using the `ResourceProfileGraphWriter` (see serialization of agent profiles).
### Hypermedia MAS Platform Profiles
To create a resource profile of a _Hypermedia MAS Platform_:
```java
ResourceProfile profile = new ResourceProfile.Builder(new HypermediaMASPlatform.Builder()
                .addHostedResource(agent)
                .addHostedResource(artifact)
                .addHostedResource(workspace)
                .setIRIAsString("https://example.org/profiles/4#platform")
                .build())
        .setIRIAsString("https://example.org/profiles/4")
        .build() ;
```
The above code snippet creates a `ResourceProfile` of a `HypermediaMASPlatform` that hosts the agent, the artifact, and the workspace previously defined. The resource profile can be serialized in Turtle using the `ResourceProfileGraphWriter` (see serialization of agent profiles).

## Interacting Through Signifiers
### Executing Actions
Signifiers exposed in resource profiles can be used to execute actions. First, we need to retrieve a signifier from a resource profile, and its signified action specification. For instance, we can retrieve an action specification based on the semantic type of an action execution:
```java
String actionType = "https://saref.etsi.org/core/ToggleCommand";
Optional<Signifier> signifier = profile.getFirstExposedSignifier(actionType);

if (signifier.isPresent()) {
  ActionSpecification actionSpec = signifier.get().getActionSpecification();
}
```

The form of the action specification can be used to prepare an action for execution. For example, if there is a registered protocol binding that is compatible with the protocol scheme of the form's target, then an `Action` object could be created and executed as follows:

```java
Form form = actionSpec.getFirstForm();
PayloadBinding payloadBinding = PayloadBindings.getBinding(form);

Action action = protocolBinding.bind(form);
ActionExecution actionExec = action.execute();
```

Alternatively, an agent can execute an action by providing an actor id that adentifies it. In the case that the built-in HTTP protocol binding is used, the actor id will be used to specify the `X-Agent-WebID` header of the generated HTTP action.

```java
String actorId = "http://hyperagent.org/alice";
ActionExecution actionExec = action.execute(actorId);
```

Additionally, if there is a registered payload binding that is compatible with the content type in the form, then an `Input` object could be created and used for executing an action. For instance, the following signifies the specification of an action for registering as the operator of an XArm robotic arm. The action requires an input whose constraints are specified in a SHACL shape (see [Artifact Profiles](#agent-profiles) for more about input specifications). The form specifies that the input should be serialized based on the `application/xarm+json` content type.

```rdf
@base <https://example.org/profiles/cherrybot> .
@prefix hctl: <https://www.w3.org/2019/wot/hypermedia#> .
@prefix htv: <http://www.w3.org/2011/http#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xs: <http://www.w3.org/2001/XMLSchema#> .
@prefix onto: <https://example.org/onto/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix hmas: <https://purl.org/hmas/> .

<#registerable> a hmas:Signifier ;
  hmas:signifies [  a sh:NodeShape;
    sh:class hmas:ActionExecution, onto:LogIn ;
    sh:property [
       sh:path prov:used ;
       sh:minCount "1"^^xs:int ;
       sh:maxCount "1"^^xs:int ;
       sh:hasValue <#form>
    ], [
      sh:path hmas:hasInput ;
      sh:qualifiedValueShape <#input-spec> ;
      sh:qualifiedMinCount 1 ;
      sh:qualifiedMaxCount 1 ;
    ], [
      sh:path hmas:hasOutput ;
      sh:qualifiedValueShape <#output-spec> ;
      sh:qualifiedMinCount 1 ;
      sh:qualifiedMaxCount 1 ;
    ]    
  ].

<#input-spec> a sh:Shape ;
  sh:class foaf:Agent ;
  sh:property [
    sh:path foaf:name ;
    sh:minCount 1;
    sh:maxCount 1 ;
    sh:datatype xs:string
  ], [
    sh:path foaf:mbox ;
    sh:minCount 1;
    sh:maxCount 1 ;
    sh:datatype xs:string
  ].

<#output-spec> a sh:Shape ;
  sh:class foaf:Agent ;
  sh:property [
    sh:path foaf:account ;
    sh:minCount 1;
    sh:maxCount 1 ;
    sh:datatype xs:string
  ] .
  
<#form> a hctl:Form;
  hctl:hasTarget <https://api.interactions.ics.unisg.ch/cherrybot/operator> ;
  htv:methodName "POST" ;
  hctl:forContentType "application/xarm+json" .
```

Based on the input specification, we can construct an input map, where the semantic annotations(s) of the specified input are used as map keys. For instance, to register as the operator of the robotic arm, the map holds the name and the mailbox address of the registering agent based on annotations defined by the [FOAF vocabulary](http://xmlns.com/foaf/spec/). Then, a payload binding that is compatible with the content type `application/xam+json` can be used to construct an input. The generated input is selialized based on the content type `application/xam+json` and validated (TBD) based on the input specification.

```java
ProtocolBinding protocolBinding = PayloadBindings.getBinding(form);

HashMap<String, String> agentDetails = new HashMap<>();
agentDetails.put("http://xmlns.com/foaf/0.1/name", "Alice");
agentDetails.put("http://xmlns.com/foaf/0.1/mbox", "alice@example.org");

Optional<IOSpecification> inputSpec = actionSpec.getInputSpecification();
if (inputSpec.isPresent()) {
  Input input = payloadBinding.bind(inputSpec, agentDetails);
  ActionExecution actionExec = action.execute(input);
}
```

In case an output specification is signified, the raw output data can be retrieved from the `ActionExecution` object. The output data can be de-serialized using the relevant payload binding.

```java
Optional<IOSpecification> outputSpec = actionSpec.getOutputSpecification();
if (outputSpec.isPresent()) {
  Map<String, Object> accountDetails = payloadBinding.unbind(outputSpec, actionExec.getOutputData());
  accountDetails.get("http://xmlns.com/foaf/0.1/account");
}
```

### Registering Custom Payload and Protocol Bidnings 
(TBA)

## Credits and Thanks

### Citation
See [the citation and DOI of the latest release](https://zenodo.org/records/14196780/latest)

### Acknowledgements
This work has received funding from the Swiss National Science Foundation under grant No. 189474 [HyperAgents](https://github.com/HyperAgents/hmas).
