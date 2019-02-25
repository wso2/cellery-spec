# Cellery Specification version 0.1.0

  [![GitHub (pre-)release](https://img.shields.io/github/release/cellery-io/spec/all.svg)](https://github.com/cellery-io/spec/releases)
  [![GitHub (Pre-)Release Date](https://img.shields.io/github/release-date-pre/cellery-io/spec.svg)](https://github.com/cellery-io/spec/releases)
  [![GitHub last commit](https://img.shields.io/github/last-commit/cellery-io/spec.svg)](https://github.com/cellery-io/spec/commits/master)
  [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## Introduction

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
* Cell
* Component
* Ingress
* Parameters
* API

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

## Cellery Language Syntax
The Cellery Language is a subset of Ballerina and Ballerina extensions. Therefore the language syntax of Cellery 
resembles with normal Ballerina language syntax.

#### Cell
A cell is a collection of components, APIs, Ingresses and Policies. A Cell object is initialized as follows;
```
cellery:CellImage cellObj = new ();
```
Components can be added to a cell by calling addComponent(Component comp) method.
```
cellObj.addComponent(comp);
```
#### Component
A component represents an implementation of the business logic (in a docker image or a legacy system) and a collections 
of network accessible entry points (Ingresses) and parameters. A sample component with inline definitions would be as follows:
```
cellery:Component stock = {
   name: "stock",
   source: {
       image: "docker.io/celleryio/sampleapp-stock"
   },
   ingresses: {
       stock: new cellery:HTTPIngress(8080,
           "stock",
           [
               {
                   path: "/options",
                   method: "GET"
               }
           ]
       )
   }
};
```

#### Ingresses
An Ingress represents an entry point into a Cell or Component. An ingress can be an HTTP or a TCP endpoint.

##### HTTP Ingresses
HTTP ingress supports defining HTTP definitions inline or as a swagger file.
A sample ingress instance  with inline API definition would be as follows:
```
cellery:Ingress ingressA = new cellery:HTTPIngress(
    8080,      //port
    "foo",        //context
     [        // Definitions
            {
                    path: "*",
                    method: "GET,POST,PUT,DELETE"
            }
        ]
};
```
A sample ingress instance  with swagger 2.0 definition would be as follows:
```
cellery:Ingress ingressA = new cellery:HTTPIngress(
    8080,
    "foo",
    “./resources/employee.swagger.json”
);
```
##### TCP Ingresses
TCP ingress supports defining TCP endpoints. A sample TCP ingress would be as following:
```
cellery:Ingress ingressA = new cellery:TCPIngress(
    8080,      // port
    9090,      // target port
};
```

The port is the actual container port which is exposed by the container. The targetPort is the port exposed by the cell gateway.

#### Parameters
A cell developer can require a set of parameters that should be passed to a Cell instance for it to be properly functional. 

```
cellery:Component employeeComponent = {
   name: "employee",
   source: {
       image: "docker.io/celleryio/sampleapp-employee"
   },
   ingresses: {
       employee: new cellery:HTTPIngress(
                     8080,
                     "employee",
                     "./resources/employee.swagger.json"
       )
   },
   parameters: {
       SALARY_HOST: new cellery:Env(),
       PORT: new cellery:Env(default = 8080)
   },
   labels: {
       cellery:TEAM:"HR"
   }
};
```

Note the parameters SALARY_HOST and PORT in the Cell definition above. This parameters can be set in the build time of this Cell:

```
public function build(string orgName, string imageName, string imageVersion) {
   // Build EmployeeCell
   io:println("Building Employee Cell ...");
   // Map component parameters
   cellery:setParameter(employeeComponent.parameters.SALARY_HOST, "http://salaryservice.com");
   // Add components to Cell
   employeeCell.addComponent(employeeComponent);
   //Expose API from Cell Gateway
   employeeCell.exposeAPIsFrom(employeeComponent);

   _ = cellery:createImage(employeeCell, orgName, imageName, imageVersion);
}
```

#### APIs
An API represents a defined set of functions and procedures that the services of the Components inside a Cell exposes 
as resources (i.e ingresses). An Ingress can be exposed as an API using exposeAPIsFrom method.
```
cellery:Component stock = {
   name: "stock",
   source: {
       image: "docker.io/celleryio/sampleapp-stock"
   },
   ingresses: {
       stock: new cellery:HTTPIngress(8080,
           "stock",
           [
               {
                   path: "/options",
                   method: "GET"
               }
           ]
       )
   }
};

cellery:CellImage stockCell = new();

public function build(string orgName, string imageName, string imageVersion) {
   //Build Stock Cell
   io:println("Building Stock Cell ...");
   stockCell.addComponent(stock);
   //Expose API from Cell Gateway
   stockCell.exposeAPIsFrom(stock);
   _ = cellery:createImage(stockCell, orgName, imageName, imageVersion);
}
```

#### Autoscaling

Autoscale policies can be specified by the Cell developer at Cell creation time. 

Cell component with scale policy at component level:

```
import ballerina/io;
import celleryio/cellery;

cellery:AutoscalingPolicy policy1 = {
   minReplicas: 1,
   maxReplicas: 2,
   metrics: [
       { new cellery:CpuUtilizationPercentage(50) }
   ]
};

//Stock Component
cellery:Component stock = {
   name: "stock",
   source: {
       image: "docker.io/celleryio/sampleapp-stock"
   },
   ingresses: {
       stock: new cellery:HTTPIngress(8080,
           "stock",
           [
               {
                   path: "/options",
                   method: "GET"
               }
           ]
       )
   },
   autoscaling: { policy: policy1, overridable: false }
};

cellery:CellImage stockCell = new();

public function build(string orgName, string imageName, string imageVersion) {
   //Build Stock Cell
   io:println("Building Stock Cell ...");
   stockCell.addComponent(stock);
   //Expose API from Cell Gateway
   stockCell.exposeAPIsFrom(stock);
   _ = cellery:createImage(stockCell, orgName, imageName, imageVersion);
}
```

The autoscale policy defined by the developer can be overriden at the runtime by providing a different policy at the runtime. 

### Intra Cell Communication

Cell components can communicate with each other. This is achieved via parameters. Two components can be linked via 
parameters in the build method. For an example, consider the scenario below. The employee component expects 
two parameters SALARY_HOST and PORT, which are the hostname and port of salary component.

Employee component:
```
cellery:Component employeeComponent = {
   name: "employee",
   source: {
       image: "docker.io/celleryio/sampleapp-employee"
   },
   ingresses: {
       employee: new cellery:HTTPIngress(
                     8080,
                     "employee",
                     "./resources/employee.swagger.json"
       )
   },
   parameters: {
       SALARY_HOST: new cellery:Env(),
       PORT: new cellery:Env(default = 8080)
   },
   labels: {
       cellery:TEAM:"HR"
   }
};
```

Salary component: 
```
cellery:Component salaryComponent = {
   name: "salary",
   source: {
       image: "docker.io/celleryio/sampleapp-salary"
   },
   ingresses: {
       SalaryAPI: new cellery:HTTPIngress(
               8080,
               "payroll",
               [{
                     path: "salary",
                     method: "GET"
               }]
           )
   }
};
```

These two parameters are provided in the build method as shown below, which enables the employee component to 
communicate with the salary component. 
```
public function build(string orgName, string imageName, string imageVersion) {
   // Build EmployeeCell
   io:println("Building Employee Cell ...");
   // Map component parameters
   cellery:setParameter(employeeComponent.parameters.SALARY_HOST, cellery:getHost(imageName, salaryComponent));
   // Add components to Cell
   employeeCell.addComponent(employeeComponent);
   //Expose API from Cell Gateway
   employeeCell.exposeAPIsFrom(employeeComponent);

   _ = cellery:createImage(employeeCell, orgName, imageName, imageVersion);
}
```

### Inter Cell Communication

In addition to components within a cell, Cells themselves can communicate with each other. This is also achieved 
via parameters. Two cells can be linked via parameters in the run method. 

When a Cell Image is built it will generate a reference file describing the APIs that are exposed by itself. 
This cell reference will be installed locally, either when you build or pull the image. This reference can be imported 
in another Cell definition which is depending on the former, and can be used to link the two Cells at the runtime. 

Sample generated reference file: 
```
# Reference to the stock cell image instance which can be used to get details about the APIs
# exposed by the the cell. This information is only relevant for calls within the cellery system.
# + instanceName - The name of the instance of the cell image to be referred to

public type StockReference object {
   string cellName = "stock";
   string cellVersion = "1.0.0";
   string instanceName = "";

   public function __init(string instanceName) {
       self.instanceName = instanceName;
   }

   # Get the Host name of the Cell Gateway. This hostname can be used when accessing the APIs.
   #
   # + return - The host name of the Cell Gateway
   public function getHost() returns string {
           return self.instanceName + "--gateway-service";
   }

   # Get the complete URL of the "stock" API exposed by the cell gateway.
   # This URL can be used when accessing the "stock" API.
   #
   # + return - The URL of the "stock" API
   public function getStockApiUrl() returns string {
       return self.getStockApiProtocol() + "://" + self.getHost() + ":"
           + self.getStockApiPort()
           + self.getStockApiContext();
   }

   # Get the protocol of the "stock" API exposed by the cell gateway.
   # This protocol can be used when accessing the "stock" API.
   #
   # + return - The protocol of the "stock" API
   public function getStockApiProtocol() returns string {
       return "http";
   }

   # Get the port which exposes the "stock" API exposed by the cell gateway.
   # This port can be used when accessing the "stock" API.
   #
   # + return - The port which exposes the "stock" API
   public function getStockApiPort() returns int {
       return 80;
   }

   # Get the context of the "stock" API exposed by the cell gateway.
   # This context path can be used when accessing the "stock" API.
   #
   # + return - The context of the "stock" API
   public function getStockApiContext() returns string {
       return "stock";
   }
};
```

If a cell component wants to access the StockAPI, it can be done as below:

```
import myorg/stock;
import ballerina/io;
import celleryio/cellery;

//HR component
cellery:Component hrComponent = {
  name: "hr",
  source: {
      image: "docker.io/wso2vick/sampleapp-hr"
  },
  ingresses: {
      hr: new cellery:HTTPIngress(8080, "info",
          [
              {
                  path: "/",
                  method: "GET"
              }
          ]
      )
  },
  parameters: {
      stockgw_url: new cellery:Env()
  }
};

// Cell Initialization
cellery:CellImage hrCell = new();

// build method for cell
public function build(string orgName, string imageName, string imageVersion) {
  // Build HR cell
  io:println("Building HR Cell ...");
  hrCell.addComponent(hrComponent);
  // Expose API from Cell Gateway & Global Gateway
  hrCell.exposeGlobalAPI(hrComponent);
  _ = cellery:createImage(hrCell);
}
```

Note the run method below, which takes a variable argument list for the references of the dependency cells. 
These are names of already deployed cell instances, which will be used to retrieve the parameters and link with this 
cell instance. As an example, the stockRef.getHost() method in the example below returns the host name of the running 
stock cell instance, which is then used to link this HR cell instance with it.
```
public function run(string imageName, string imageVersion, 
string instanceName, string... dependenciesRef) {
   stock:StockReference stockRef = new(dependenciesRef[0]);
   cellery:setParameter(hrComponent.parameters.stockgw_url, stockRef.getHost());
   hrCell.addComponent(hrComponent);
   _ = cellery:createInstance(hrCell, imageName, imageVersion, instanceName);
}
```
