# Environments

An Environment describes a configuration tailored for a context of any kind
whatsoever.
Such a context might be `dev` and `prod` environments, blue/green deployments or the
same application available in multiple regional zones / datacenters.

An environment does not need to be created, it rather just exists as soon as a
main.jsonnet is found somewhere in the tree of a `rootDir`. This also means that
an environment by definition is equivalent to a [`baseDir`](directory-structure.md#base-directory-basedir).

## Configuration
To correctly deal with an environment, Tanka needs some additional information
about it. These are specified in a file called `spec.json` which is placed next to
`main.jsonnet`.

```json
{
  "apiVersion": "tanka.dev/v1alpha1",
  "kind": "Environment",
  "metadata": {
    "name": "auto",
    "labels": {}
  },
  "spec": {
    "apiServer": "https://localhost:6443",
    "clusterName": "",
    "namespace": "default"
  }
}
```

| Field                  | Description                                                          |
|------------------------|----------------------------------------------------------------------|
| `apiVersion`           | currently only `tanka.dev/v1alpha1` is available                     |
| `kind`                 | always `Environment`                                                 |
| `metadata.name`        | *automatically set to the directory name*                            |
| `metadata.labels`      | descriptive `key:value` pairs                                        |
| **`spec.clusterName`** | The Kubeconfig cluster name to use. Optional if apiServer is defined |
| **`spec.apiServer`**   | The Kubernetes endpoint to use, overriding clusterName               |
| **`spec.namespace`**   | All objects will be created in this namespace                        |

Everything written in **bold** is required, the other fields may be omitted.

## Context discovery
To make sure you **never** apply to the wrong cluster, Tanka parses the output
of `kubectl config view` to select a context that matches the API Server
endpoint specified in the [Configuration](#configuration).

If `spec.clusterName` is set, Tanka first searches for a `cluster` whose name
matches the configured name followed by a context using that cluster. Otherwise,
Tanka searches for a `cluster` that matches the IP or hostname specified in
`spec.apiServer` and a context using that cluster. If the cluster or context
could not be found, the apply will fail.

So please make sure `$KUBECONFIG` and `kubectl` are set up correctly if you run
into any problems.
