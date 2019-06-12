# Cellery Specification version 0.2.0

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
* Dependencies

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
environment variable, secret or a configuration map. Environment variables can be passed into a cell runtime via two ways as given below.  

1) With `-e` inline param when running the cell (e.g: cellery run wso2cellery/test:1.0.0 -e myEnv=test)
2) Any environment variable starts with `CELLERY`. In that case, users do not need to pass it via inline variable, they 
can simply export the intended environment variable it and run the cell.

#### API
An API represents a defined set of functions and procedures that the services of the Components inside a Cell exposes 
as resources (i.e ingresses). These could be made accessible internally or Globally.

#### Dependencies
A cell can depend on other cells. The components defines a set of cells that it depends on. 
Celley ensures that the dependencies are met at the runtime. 

## Cellery Language Syntax
The Cellery Language is a subset of Ballerina and Ballerina extensions. Therefore the language syntax of Cellery 
resembles with normal Ballerina language syntax.

#### Cell
A Cell is a collection of components, APIs, Ingresses and Policies. A Cell record is initialized as follows;
Components can be added to the components map in the CellImage record.
```ballerina
cellery:CellImage helloCell = {
   components: {
       helloComp:helloWorldComp
   }
};

```

#### Component
A component represents an implementation of the business logic (in a docker image or a legacy system) and a collections 
of network accessible entry points (Ingresses) and parameters. A sample component with inline definitions would be as follows:
```ballerina
cellery:Component helloComponent = {
    name: "hello-api",
    source: {
        image: "docker.io/wso2cellery/samples-hello-world-api" // source docker image
    },
    ingresses: {
        helloApi: <cellery:HttpApiIngress>{ 
            port: 9090,
            context: "hello",
            definition: {
                resources: [
                    {
                        path: "/",
                        method: "GET"
                    }
                ]
            },
            expose: "global"
        }
    },
    envVars: {
        MESSAGE: { value: "hello" }
    }
};
```
#### Ingresses
An Ingress represents an entry point into a cell component. An ingress can be an HTTP, TCP, GRPC or Web endpoint.

##### 1. HTTP Ingresses
`HttpApiIngress` supports defining HTTP API as an entry point for a cell. API definitions can be provided inline or as a swagger file. 
A sample `HttpApiIngress` instance  with inline API definition would be as follows:
```ballerina
cellery:HttpApiIngress helloAPI = {
    port: 9090,
    context: "hello",
    definition: {
        resources: [
            {
                path: "/",
                method: "GET"
            }
        ]
    },
    expose: "global"
};

```
A sample `HttpApiIngress` record  with swagger 2.0 definition can be defined as follows. The definitions are resolved at the build time. 
Therefore the build method is implemented to parse swagger file, and assign to ingress. 
```ballerina
cellery:HttpApiIngress employeeIngress = {
    port: 8080,
    context: "employee",
    expose: "local"
};

public function build(cellery:ImageName iName) returns error? {
    cellery:HttpApiIngress helloAPI = {
        port: 9090,
        context: "hello",
        definition: <cellery:ApiDefinition>cellery:readSwaggerFile("./resources/employee.swagger.json"),
        expose: "global"
    };
    ...
}
```

###### Expose
An `HttpApiIngress` can be exposed as an API by setting `expose` field. This field accepts two values.

    -  `local`: Expose an HTTP API via local cell gateway.
    -  `global`: Expose an HTTP API via global gateway.


##### 2. Web Ingresses
Web cell ingress allows web traffic to the cell. A sample Web ingress would be as following: 
Web ingress are always exposed globally.

```ballerina
cellery:WebIngress webIngress = { 
    port: 8080,
    gatewayConfig: {
        vhost: "abc.com",
        context: "/demo" //default to “/”
    }
};
```
###### 2.1 Define TLS for Web Ingress
TLS can be defined to a web ingress as below:
```ballerina
// Web Component
cellery:Component webComponent = {
    name: "web-ui",
    source: {
        image: "wso2cellery/samples-hello-world-webapp"
    },
    ingresses: { 
        webUI: <cellery:WebIngress>{ // Ingress is defined in line in the ingresses map.
            port: 80,
            gatewayConfig: {
                vhost: "hello.com",
                tls: {
                    key: "",
                    cert: ""
                }
            }
        }
    }
};
// Create the cell image with cell web component.
cellery:CellImage webCell = {
      components: {
          webComp: webComponent
      }
  };

```
Values for tls key and tls cert can be assigned at the run method as below.
```ballerina
public function run(cellery:ImageName iName, map<cellery:ImageName> instance) returns error? {
    //Read TLS key file path from ENV and get the value
    string tlsKey = readFile(config:getAsString("tls.key"));
    string tlsCert = readFile(config:getAsString("tls.cert"));

    //Assign values to cell->component->ingress
    cellery:CellImage webCell = check cellery:constructCellImage(untaint iName);
    cellery:WebIngress webUI = <cellery:WebIngress>webCell.components.webComp.ingresses.webUI;
    webUI.gatewayConfig.tls.key = tlsKey;
    webUI.gatewayConfig.tls.cert = tlsCert;
    // Create the cell instance
    return cellery:createInstance(webCell, iName);
}

// Read the file given in filePath and return the content as a string.
function readFile(string filePath) returns (string) {
    io:ReadableByteChannel bchannel = io:openReadableFile(filePath);
    io:ReadableCharacterChannel cChannel = new io:ReadableCharacterChannel(bchannel, "UTF-8");

    var readOutput = cChannel.read(2000);
    if (readOutput is string) {
        return readOutput;
    } else {
        error err = error("Unable to read file " + filePath);
        panic err;
    }
}
```
###### 2.2 Authenticate Web Ingress
Web ingress support Open ID connect. OIDC config can be defined as below.
```ballerina
cellery:Component portalComponent = {
    name: "portal",
    source: {
        image: "wso2cellery/samples-pet-store-portal"
    },
    ingresses: {
        portal: <cellery:WebIngress>{ // Web ingress will be always exposed globally.
            port: 80,
            gatewayConfig: {
                vhost: "pet-store.com",
                context: "/",
                oidc: {
                    nonSecurePaths: ["/", "/app/*"],
                    providerUrl: "",
                    clientId: "",
                    clientSecret: "",
                    redirectUrl: "http://pet-store.com/_auth/callback",
                    baseUrl: "http://pet-store.com/",
                    subjectClaim: "given_name"
                }
            }
        }
    }
};
```

If dynamic client registration is used dcr configs can be provided as below in the `clientSecret` field.
```ballerina
    ingresses: {
        portal: <cellery:WebIngress>{
            port: 80,
            gatewayConfig: {
                vhost: "pet-store.com",
                context: "/portal",
                oidc: {
                    nonSecurePaths: ["/portal"], // Default [], optional field
                    providerUrl: "https://idp.cellery-system/oauth2/token",
                    clientId: "petstoreapplicationcelleryizza",
                    clientSecret: {
                        dcrUser: "admin",
                        dcrPassword: "admin"
                    },
                    redirectUrl: "http://pet-store.com/_auth/callback",
                    baseUrl: "http://pet-store.com/items/",
                    subjectClaim: "given_name"
                }
            }
        }
    },
```
Similar to above sample the `clientSecret` and `clientId` values be set at the run method to pass value at the run time without burning to the image.

##### 3. TCP Ingresses
TCP ingress supports defining TCP endpoints. A sample TCP ingress would be as following:
```ballerina
cellery:TCPIngress tcpIngress = {
    backendPort: 3306,
    gatewayPort: 31406
};
```

The backendPort is the actual container port which is exposed by the container. The gatewayPort is the port exposed by the cell gateway.

##### 4. GRPC Ingresses
GRPC ingress supports defining GRPC endpoints. This is similar to TCP ingress with optional field to define protofile. 
protofile field is resolved at build method since protofile is packed at build time.
```ballerina
cellery:GRPCIngress grpcIngress = {
    backendPort: 3306,
    gatewayPort: 31406
};

public function build(cellery:ImageName iName) returns error? {                                                                    
    grpcIngress.protoFile = "./resources/employee.proto";
    ...
}
```

#### EnvVars
A cell developer can require a set of environment parameters that should be passed to a Cell instance for it to be properly functional. 

```ballerina
// Employee Component
cellery:Component employeeComponent = {
    name: "employee",
    source: {
        image: "docker.io/celleryio/sampleapp-employee"
    },
    ingresses: {
        employee: <cellery:HttpApiIngress>{
            port: 8080,
            context: "employee",
            expose: "local"
        }
    },
    envVars: {
        SALARY_HOST: { value: "" }
    },
    labels: {
        team: "HR"
    }
};
```

Note the parameters SALARY_HOST in the Cell definition above. This parameters can be set in the run time of this Cell. 

```ballerina
public function run(cellery:ImageName iName, map<cellery:ImageName> instances) returns error? {
    employeeCell.components.empComp.envVars.SALARY_HOST.value = config:getAsString("salary.host");
    return cellery:createInstance(employeeCell, iName);
}
```

#### Autoscaling

Autoscale policies can be specified by the Cell developer at Cell creation time. 

Cell component with scale policy at component level:

```ballerina
import ballerina/io;
import celleryio/cellery;

cellery:Component petComponent = {
    name: "pet-service",
    source: {
        image: "docker.io/isurulucky/pet-service"
    },
    ingresses: {
        stock: <cellery:HttpApiIngress>{ port: 9090,
            context: "petsvc",
            definition: {
                resources: [
                    {
                        path: "/*",
                        method: "GET"
                    }
                ]
            }
        }
    },
    autoscaling: {
        policy: {
            minReplicas: 1,
            maxReplicas: 10,
            cpuPercentage: <cellery:CpuUtilizationPercentage>{ percentage: 50 }
        }
    }
};
```

The autoscale policy defined by the developer can be overriden at the runtime by providing a different policy at the runtime. 

### Intra Cell Communication

Cell components can communicate with each other. This is achieved via environment variables. Two components can be linked via 
environment variables in the run method. For an example, consider the scenario below. The employee component expects 
`SALARY_HOST`, which is the hostname of salary component.

Employee component:
```ballerina
import ballerina/io;
import celleryio/cellery;

public function build(cellery:ImageName iName) returns error? {
    int salaryContainerPort = 8080;

    // Salary Component
    cellery:Component salaryComponent = {
        name: "salary",
        source: {
            image: "docker.io/celleryio/sampleapp-salary"
        },
        ingresses: {
            SalaryAPI: <cellery:HttpApiIngress>{
                port:salaryContainerPort,
                context: "payroll",
                definition: {
                    resources: [
                        {
                            path: "salary",
                            method: "GET"
                        }
                    ]
                },
                expose: "local"
            }
        },
        labels: {
            team: "Finance",
            owner: "Alice"
        }
    };

    // Employee Component
    cellery:Component employeeComponent = {
        name: "employee",
        source: {
            image: "docker.io/celleryio/sampleapp-employee"
        },
        ingresses: {
            employee: <cellery:HttpApiIngress>{
                port: 8080,
                context: "employee",
                expose: "local",
                definition: <cellery:ApiDefinition>cellery:readSwaggerFile("./resources/employee.swagger.json")
            }
        },
        envVars: {
            SALARY_HOST: {
                value: cellery:getHost(salaryComponent)
            },
            PORT: {
                value: salaryContainerPort
            }
        },
        labels: {
            team: "HR"
        }
    };

    cellery:CellImage employeeCell = {
        components: {
            empComp: employeeComponent,
            salaryComp: salaryComponent
        }
    };

    return cellery:createImage(employeeCell, untaint iName);
}
```

The envVar `SALARY_HOST` value is provided at the build time as shown above, which enables the employee component to 
communicate with the salary component. 


### Inter Cell Communication

In addition to components within a cell, Cells themselves can communicate with each other. This is also achieved 
via envVars. Two cells can be linked via envVar in the run method. 

When a Cell Image is built it will generate a reference file describing the APIs/Ports that are exposed by itself. 
This cell reference will be available locally, either when you build or pull the image. This reference can be imported 
in another Cell definition which is depending on the former, and can be used to link the two Cells at the runtime. 

Consider following cell definition:
```ballerina
import ballerina/io;
import celleryio/cellery;

public function build(cellery:ImageName iName) returns error? {
    //Build Stock Cell
    io:println("Building Stock Cell ...");
    //Stock Component
    cellery:Component stockComponent = {
        name: "stock",
        source: {
            image: "docker.io/celleryio/sampleapp-stock"
        },
        ingresses: {
            stock: <cellery:HttpApiIngress>{ port: 8080,
                context: "stock",
                definition: {
                    resources: [
                        {
                            path: "/options",
                            method: "GET"
                        }
                    ]
                },
                expose: "local"
            }
        }
    };
    cellery:CellImage stockCell = {
        components: {
            stockComp: stockComponent
        }
    };
    return cellery:createImage(stockCell, untaint iName);
}
```

Generated reference file for above cell definition is as follows: 
```json
{
  "stock_api_url":"http://{{instance_name}}--gateway-service:80/stock"
}
```

If a cell component wants to access the stock api, it can be done as below:

```ballerina
public function build(cellery:ImageName iName) returns error? {
    //HR component
    cellery:Component hrComponent = {
        name: "hr",
        source: {
            image: "docker.io/celleryio/sampleapp-hr"
        },
        ingresses: {
            "hr": <cellery:HttpApiIngress>{
                port: 8080,
                context: "hr-api",
                definition: {
                    resources: [
                        {
                            path: "/",
                            method: "GET"
                        }
                    ]
                },
                expose: "global"
            }
        },
        envVars: {
            stock_api_url: { value: "" }
        },
        dependencies: {
            cells: {
                stockCellDep: <cellery:ImageName>{ org: "myorg", name: "stock", ver: "1.0.0" } // dependency as a struct
            }
        }
    };

    hrComponent.envVars = {
        stock_api_url: { value: <string>cellery:getReference(hrComponent, "stockCellDep").stock_api_url }
    };

    // Cell Initialization
    cellery:CellImage hrCell = {
        components: {
            hrComp: hrComponent
        }
    };
    return cellery:createImage(hrCell, untaint iName);
}

public function run(cellery:ImageName iName, map<cellery:ImageName> instances) returns error? {
    cellery:CellImage hrCell = check cellery:constructCellImage(untaint iName);
    return cellery:createInstance(hrCell, iName, instances);
}
```
 
The `hrComponent` depends on the stockCell that is defined earlier. The dependency information are specified as a component attribute.
```ballerina
    dependencies: {
        cells: {
            stockCellDep: <cellery:ImageName>{ org: "myorg", name: "stock", ver: "1.0.0" } // dependency as a struct
        }
    }
```

`cellery:getReference(hrComponent, "stockCellDep").stock_api_url` method reads the json file and set value with place holder. 

Note the `run` method above, which takes a variable argument map for the references of the dependency cells. 
These are names of already deployed cell instances, which will be used to resolve the urls and link with this 
cell instance. As an example, if the stock cell instance name is `stock-app` then the `stockRef.stock_api_url` 
returns the host name of the running stock cell instance as `http://stock-app--gateway-service:80/stock`.
