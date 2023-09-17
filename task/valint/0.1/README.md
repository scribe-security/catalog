# Valint

Valint is a tool used to manage `evidence` generation (for directories, file artifacts, images, and git repositories), storage and validation. Valint currently supports two types of evidence: **CycloneDX SBOMs** and **SLSA provenance**. It enables cryptographically signing the evidence generated allowing you to later verify artifacts against their origin and signer identity. 

Valint also enables you to **capture** any 3rd party report, scan or configuration (any file) into evidence. 

## Installing the Task

```bash
kubectl apply -f https://api.hub.tekton.dev/v1/resource/tekton/task/scribe-security/0.1/raw
```

## Storing your credentials

The `valint` task looks for a Kubernetes secret that stores your Scribe user credentials. This secret is called `scribe-secret` by default and is expected to have the keys `scribe-client-id` and `scribe-client-secret`.
You can use the following example configuration. Make sure to provide the correct credentials for your Scribe environment.

```yaml
# scribe-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: orka-creds
type: Opaque
stringData:
  scribe_client_id: $(client_id)
  scribe_client_secret: $(client_secret)
  scribe_enable: true
```

```sh
kubectl apply --namespace=<namespace> -f scribe-secret.yaml
```

Omit `--namespace` if installing in the `default` namespace.

> **NOTE:** These credentials are used by the `valint` task to generate an authentication token to access the Scribe API.

## Parameters

| Parameter | Description | Default |
| --- | --- | ---: |
| `scribe-secret` | The name of the secret that has the scribe security secrets. | scribe-secret |
| `args` | Arguments of the `valint` CLI | |
| `image-version-sha` | The ID of the valint image cli to be used. | |

## Platforms

The Task can be run on `linux/amd64` platforms.

## Usage

This TaskRun runs the Task to generate sbom for images, files, directories or git repositories.

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: valint-task-run-bom
spec:
  taskRef:
    name: valint
  params:
    - name: args
      value: 
        - bom 
        - alpine:latest
```

This task can also run the task to verify policies,

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: valint-bom-verify
spec:
  taskRef:
    name: valint
  params:
    - name: args
      value: 
        - bom
        - alpine:latest
        - -o=statement

  taskRef:
    name: valint
  params:
    - name: args
      value: 
        - verify
        - alpine:latest
        - -i=statement
```

## SLSA Usage

This task can also run the task generate and verify SLSA Provenance for images, files, directories or git repositories.

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: valint-slsa-prov-verify
spec:
  taskRef:
    name: valint
  params:
    - name: args
      value: 
        - slsa
        - alpine:latest
        - -o=statement

  taskRef:
    name: valint
  params:
    - name: args
      value: 
        - verify
        - alpine:latest
        - -i=statement-slsa
```
