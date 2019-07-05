# Cellery Specification version 0.3.0

  [![GitHub (pre-)release](https://img.shields.io/github/release/cellery-io/spec/all.svg)](https://github.com/cellery-io/spec/releases)
  [![GitHub (Pre-)Release Date](https://img.shields.io/github/release-date-pre/cellery-io/spec.svg)](https://github.com/cellery-io/spec/releases)
  [![GitHub last commit](https://img.shields.io/github/last-commit/cellery-io/spec.svg)](https://github.com/cellery-io/spec/commits/master)
  [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

### What is Cellery?
Cellery is an architectural description language, tooling and extensions for popular cloud-native runtimes that enables 
agile development teams to easily create a composable enterprise in a technology neutral way.

### How does Cellery work? 
The Cellery Language is a subset of Ballerina and Ballerina extensions. The language allows developers to define cells 
in a simple, effective and compilation-validated way. The language has an opinionated view of a composable enterprise, 
with well-defined concepts of control plane, data plane, ingress/egress management and observability.  
 
The Cellery Language is completely technology neutral. It supports any container runtimes,
cloud orchestration model, control plane, data plane and gateway. It also enables existing legacy systems as well as
external systems (including SaaS and partner APIs) to be described as cells.  

The Cellery tooling compiles and validates the cell definitions written in the Cellery Language. The tool generates an 
immutable, versioned, compiled representation of the input language, which is called a Cell Image. This file refers 
to external dependencies; especially container images and it bakes in the specific image ids so that a given version 
of a cell image will deploy in a consistent immutable way forever.  

The deployment definitions works for cloud-native runtimes such as Kubernetes. It also creates well-defined visual 
representations of the cell. Further tooling will potentially be able to extrapolate the cell 
interface and dependencies completely. The tooling is a key part of CI/CD pipelines as it can also help build external 
dependencies such as Dockerfiles.  

The Cellery language understands that cloud native applications are deployed in multiple environments from integration 
test through to staging and production, and also aids in patterns such as Blue/Green and Canary deployment.
The Cellery runtime extensions for different cloud native orchestration systems, enable those systems to directly 
understand the concept of a cell. In addition, observability allow a cell to be monitored, logged, and traced.  

Cellery can be used standalone, but over time the project will also create a repository manager for cell definitions 
that can be deployed inside an organisation, run in the cloud and in hybrid scenarios (a.k.a Cellery Central).

### What would the project contain (initially)?
* A specification of the Cellery Language: Cellery Language is a subset (and/or extension) of Ballerina that allows you 
to define Cells.
* The Cellery compiler: This validates and compiles Cellery into deployable artifacts.
* The Cellery runtime extensions for Kubernetes: A set of extensions for K8s including CRDs and Admission Controllers, 
observability plugins etc. 
* A repository for cell definitions: e.g. Cellery Central to allow push/pull of cell definitions.

### How does Cellery compare to existing approaches?
Cellery overlaps with Docker Compose, Helm and Kubernetes YAML (plus….) Comparison points:
* It only generates deployment artefacts so it works with existing runtimes. Runtime extensions can improve this default behaviour.
* Cellery has an opinionated view of what a composable architecture should look like.
* Compilation ensures development time validation of links between containers in a way that those other approaches do not have.

### Concepts of Cellery
Following are the key concepts of Cellery:
* [Cell](#cell) 
* [Component](#component)
* [Ingress](#ingress)
* [Parameters](#parameters)
* [API](#api)
* [Dependencies](#dependencies)
* [Resources](#resources)
* [Probes](#probes)
* [Scaling](#scaling)

#### Cell
A cell is a collection of components, grouped from design and implementation into deployment. A cell is independently 
deployable, manageable, and observable.  

Components inside the cell can communicate with each other using supported transports for intra-cell communication. 
External communication must happen through the edge-gateway or proxy, which provides APIs, events, or streams via 
governed network endpoints using standard network protocols.  

A cell can have 1:n components grouped. Components inside the cells are reusable and can be instantiated in multiple 
cells. The cell should document its offers. The capabilities of a cell must be provided by network accessible endpoints. 
In addition, if the cell needs access to external dependencies, then these must also be exposed as network endpoints 
through a cell-gateway. These endpoints can expose APIs, events, or streams. Any interfaces that the microservices or 
serverless components offer that are not made available by the control point should be inaccessible from outside the 
cell. Every component within the cell should be versioned. The cell should have a name and a version identifier. 
The versions should change when the cell’s requirements and/or offers change.

#### Component
A component represents business logic running in a container, serverless environment, or an existing runtime. 
This can then be categorized into many subtypes based on the functional capabilities. A component is designed based 
on a specific scope, which can be independently run and reused at the runtime. Runtime requirements and the behavior of 
the component vary based on the component type and the functional capabilities. The user may decide to build and run 
the code as a service, function, or microservice, or choose to reuse an existing legacy service based on the architectural need.

#### Ingress
An Ingress represents an entry point into a Cell or Component. These can expose APIs, events, or streams. 
Any interfaces that the components offer that are not made available by the control point should be inaccessible from outside the cell.

#### Parameters
A parameter represents an external entity that a particular component require to operate. The parameter can be a 
environment variable, secret or a configuration map. 

#### API
An API represents a defined set of functions and procedures that the services of the Components inside a Cell exposes 
as resources (i.e ingresses). These could be made accessible internally or Globally.

#### Dependencies
A component can depend on other cells. The components defines a set of cells/components that it depends on. 
Cellery ensures that the dependencies are met at the runtime.

#### Resources
Cellery allows to specify how much CPU and memory (RAM) each component needs. Resources can be specified as requests and limits. 
When component have resource requests specified, the scheduler can make better decisions about which nodes to place component on.
And when component have their limits specified, contention for resources on a node can be handled in a specified manner. 

#### Probes
Liveness and readiness probes can be defined for a component. The probes can be defined by the means of command, http-get or as a tcp socket.

#### Scaling 
Each component within the cells can be scaled up or down. Cellery supports auto scaling with Horizontal pod autoscaler, 
and Zero-scaling. 

## What's Next?
- Check [Cellery syntax](https://github.com/wso2-cellery/sdk/blob/master/docs/cellery-syntax.md) which is based on this specification.