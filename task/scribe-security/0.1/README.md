# Generate SBOM and Verify SBOM

This task will help scribe security users to generate sbom and verify the sbom using tekton

# Generate SBOM

This task uses the scribe security valint image that can execute the valint binary

## Install the Task and create a secret

```
kubectl apply -f https://api.hub.tekton.dev/v1/resource/tekton/task/scribe-security/0.1/raw
```

Create a secret that has client_id, client_secret, product_id.

Example scribe-security-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: scribe-secret
  annotations:
    tekton.dev/git-0: https://github.com
    tekton.dev/git-1: https://gitlab.com
    tekton.dev/docker-0: https://gcr.io
type: Opaque
stringData:
  product_key: $(product_key)
  client_id: $(client_id)
  client_secret: $(client_secret)

```

Example kubectl command
```
kubectl apply -f scribe-security-secret.yaml
```

## Parameters

* **scribe-secret**: The name of the secret that has the scribe security secrets.

* **args**: the arguments of the `valint` CLI, which are appended 

* **image-version-sha**: The ID of the valint image cli to be used.

## Platforms

The Task can be run on `linux/amd64` platforms.

## Usage

This TaskRun runs the Task to generate sbom for images, files and compressed files

```
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: scribe-security-task-run-10
spec:
  taskRef:
    name: scribe-security
  params:
    - name: args
      value: bom alpine:latest
    - name: scribe-secret
      value: scribe-secret
```

This task can also run the task to verify the sbom.

```
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: scribe-security-task-run-10
spec:
  taskRef:
    name: scribe-security
  params:
    - name: args
      value: verify alpine:latest
    - name: scribe-secret
      value: scribe-secret
```
