# hmas-java

A library for handling resources in Hypermedia Multi-Agent Systems based on the HyperAgents ontologies:
- [Hypermedia MAS Core Ontology](https://purl.org/hmas/core) (see [core](https://github.com/danaivach/hmas-core))
- [Hypermedia MAS Interaction Ontology](https://purl.org/hmas/interaction) (see [interaction](https://github.com/danaivach/hmas-interaction))

### Table of Contents
- Getting Started
- Retrieving and Parsing Resource Profiles
- Creating and Writing Resource Profiles
  - Agent Profiles
  - Artifact Profiles
  - Workspace Profiles
  - Hypermedia MAS Platform Profiles

## Getting Started
TBA

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
ArtifactProfile profile = ArtifactProfileGraphReader.readFromString(TDFormat.RDF_TURTLE, artifactProfile);
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

ArtifactProfile profile =
            new ArtifactProfile.Builder(artifact)
                    .setIRIAsString("https://example.org/profiles/2")
                    .exposeSignifier(togglable);
                    .build();
```
The above code snippet creates an `ArtifactProfile` of an `saref:LightSwitch` `Artifact` (see the [SAREF Ontology](https://saref.etsi.org/core/v3.1.1/)https://saref.etsi.org/core/v3.1.1/). The profile exposes a `Signifier` for indicating that the light switch is `togglable`. This Signifier can be defined in the following manner:
```java
Signifier signifier = new Signifier.Builder(new ActionSpecification.Builder(toggleForm)
                            .addSemanticType("https://saref.etsi.org/core/ToggleCommand")
                            .setRequiredInput(inputSpecification)
                            .build())
                    .build()   
```
This signifier signifies an `ActionSpecification` of an action that has the semantic type `saref:ToggleCommand`. The action requires an input defined in an `inputSpecification`, and is implemented through a `toggleForm` `Form`, which is a type of hypermedia control.

A required input can be defined in an `InputSpecification` that defines contraints on the expected input in the form of a [SHACL shape](https://www.w3.org/TR/shacl/#shapes). For example, to specify that the input should be a `saref:State` that `saref:hasValue` an `integer` value: 
```java
InputSpecification inputSpecification = new InputSpecification.Builder()
        .withRequiredSemanticType("https://saref.etsi.org/core/State")
        .withProperty(new PropertySpecification.Builder("https://saref.etsi.org/core/hasValue")
                .withDataType("http://www.w3.org/2001/XMLSchema#integer")
                .withMinCount(1)
                .withMaxCount(1)
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
String artifactProfile = new ArtifactProfileGraphWriter(profile)
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
      sh:qualifiedValueShape [ a sh:NodeShape;
        sh:class saref:State;
        sh:property [
          sh:path saref:hasValue;
          sh:minCount "1"^^xs:int;
          sh:maxCount "1"^^xs:int;
          sh:datatype xs:integer
        ] 
      ] ;
      sh:qualifiedMinCount "1"^^xs:int;
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
