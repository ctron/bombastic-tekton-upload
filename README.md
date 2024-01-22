# Bombastic SBOM upload example with Tekton

## Pre-requisites

* A working cluster
* Logged in (regular user)
* A working Tekton installation
* A working Trustification/RHTPA installation
* Installed `tkn` CLI

## Assumptions

* You're running OCP and use the `oc` CLI. It is possible to switch to `kubectl` too.
* Trustification is installed in the namespace `trustification`, and you have access to read its `Secret`s.
* You're using the namespace `test`. You can use any other namespace, just replace the namespace arguments and values.
* All commands are executed from the directory of this document
* You have access to the "walker" OAuth2 credentials. You can get them by executing the following commands:
  
  ```shell
  oc -n trustification get secrets oidc-client-walker -o 'jsonpath={ .data.client-secret }' | base64 -d 
  ```

## Installation

First, we need to install the task itself:

```shell
oc -n test apply -f install
```

## Example

A quick example with mock data.

### Setup

Next, we create a task which generates us an SBOM for testing:

```shell
oc -n test apply -f examples/task-create.yaml
```

Then, we create an example pipeline:

```shell
oc -n test apply -f examples/pipeline.yaml
```

### Running via the CLI

Then, we can trigger a run of this pipeline. It is necessary to provide some parameters:

```shell
tkn pipeline start example -n test -w 'name=sboms,volumeClaimTemplateFile=examples/workspace-volume-claim.yaml' -p bombastic-api-url=https://sbom.apps.cluster-8mn59.sandbox909.opentlc.com/api/v1/sbom -p bombastic-issuer-url=https://sso.apps.cluster-8mn59.sandbox909.opentlc.com/realms/chicken -p bombastic-client-id=walker -p bombastic-client-secret=$(oc -n trustification get secrets oidc-client-walker -o 'jsonpath={ .data.client-secret }' | base64 -d)
```

### Following the logs

Run:

```shell
tkn p logs -L -f example
```