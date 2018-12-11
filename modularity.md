# Modularity & Versioning

Cellery programs can be written in one or more files organized into packages.

## Files

A Cellery file is structured as follows:
```
[import PackageName [version ImportVersionNumber] [as Identifier];]*
import celleryio/cellery;

(ComponentDefinition | CellInitialization)+

lifeCycleBuild()
```

A sample file would be as follows;
```
import ballerina/io;
import celleryio/cellery;

//Components
cellery:Component componentA = {
    name: "ComponentA",
    source: {
        image: "docker.io/wso2vick/component-a:v1"
    },
    env: { ENV1: "", ENV2: "" },
    ingresses: [
        {
            name: "foo",
            port: "8080:80",
            context: "foo",
            definitions: [
                {
                    path: "*",
                    method: "GET,POST,PUT,DELETE"
                }
            ]
        }
    ],
    egresses: [
        {
            parent: componentB.name,
            ingress: componentB.ingresses[0],
            envVar: "ENV1",
            resiliency: {
                retryConfig: {
                    interval: 100,
                    count: 10,
                    backOffFactor: 0.5,
                    maxWaitInterval: 20000
                }
            }
        }
    ]
};

cellery:Component componentB = {
    name: "ComponentB",
    source: {
        image: "docker.io/wso2vick/component-b:v1"
    },
    env: { ENV1: "", ENV2: "" },
    ingresses: [
        {
            name: "bar",
            port: "8080:80",
            context: "bar",
            definitions: [
                {
                    path: "*",
                    method: "GET,POST,PUT,DELETE"
                }
            ]
        }
    ]
};

//Cell
cellery:Cell cellA = new;

//Build Function
public function lifeCycleBuild() {
    componentA.env["ENV2"] = "1VALUE2";
    cellA.addComponent(componentA);

    componentB.env["ENV1"] = "2VALUE1";
    componentB.env["ENV2"] = "2VALUE2";
    cellA.addComponent(componentB);

    cellA.egresses = [
        {
            parent: componentB.name,
            ingress: componentB.ingresses[0],
            envVar: "ENV1"
        }
    ];

    cellA.apis = [
        {
            parent: componentB.name,
            context:componentB.ingresses[0],
            global: true
        }
    ];

    io:println(cellA);
    var v = cellery:build(cellA);
}
```

## Versioning

