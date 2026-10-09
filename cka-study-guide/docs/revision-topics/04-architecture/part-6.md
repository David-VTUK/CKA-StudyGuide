# Use Helm and Kustomize to install cluster components

## Helm

Helm is synonymous to what `apt` or `yum` are in the Linux world. It's effectively a package manager for Kubernetes. "Packages" in Helm are called `charts` to which you can customise with your own values.

It's unlikely the exam will require anyone to create a helm chart from scratch, but an understanding of how it works is a good idea.

### Helm Repos

Repos are where helm charts are stored. Typically, a repo will contain a number of charts to choose from. Helm can be managed by a CLI client, and a repo can be added by running:

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

To list the packages from this repo:

```shell
helm search repo bitnami
```

To install a package from this repo:

```shell
helm install my-release bitnami/mariadb
```

Where `my-release` is a string identifying an installed instance of this application

Parameters that can be customised - are dependent on how the chart is configured. For the aforementioned MariaDB chart, they are listed at [https://github.com/bitnami/charts/tree/master/bitnami/mariadb/#parameters](https://github.com/bitnami/charts/tree/master/bitnami/mariadb/#parameters)

These values are encapsulated in the corresponding `values.yaml` file in the repo. You can populate an instance of it and apply it with:

```shell
helm install -f https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mariadb/values.yaml my-release bitnami/mariadb
```

Alternatively, variables can be declared by using `--set`, such as:

```shell
helm install my-release --set auth.rootPassword=secretpassword bitnami/mariadb
```

## Kustomize

Kustomize is a templating tool for Kubernetes manifests in its native form (Yaml). When working with raw YAML files you will typically have a directory containing several files identifying the resources it creates. To begin, a directory containing our manifests needs to exist:

```shell
/home/david/app/base
total 16
drwxrwxr-x  2 david david 4096 Feb  9 11:44 .
drwxr-xr-x 27 david david 4096 Feb  9 11:44 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

This will form our `base` - we will build on this but adding customisations in the form of overlays. First, we need a `kustomize` file. which can be created with `kustomize create --autodetect`

This will create kustomization.yaml in the current directory:

```shell
total 20
drwxrwxr-x  2 david david 4096 Feb  9 11:47 .
drwxr-xr-x 27 david david 4096 Feb  9 11:47 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  108 Feb  9 11:47 kustomization.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

The contents being:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
```

### Variants and Overlays

* variant - Divergence in configuration from the `base`
* overlay - Composes variants together

Say, for example, we wanted to generate manifests for different environments (prod and dev) that are based from this config, but have additional customisations. In this example we will create a `dev` variant encapsulated in a single Overlay

```shell
mkdir -p overlays/{dev,prod}
cd overlays/dev 
```

Begin by creating a Kustomization object specifying the base (this will create `kustomization.yaml`) :

```shell
kustomize create --resources ../../base
```

In this example, I want to change the replica count to 1, as it's a dev environment. In the `dev` directory, create a new file `deployment.yaml` containing:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
```

The `kustomization.yaml` file needs modifying to include a `patchesStrategicMerge` block:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
patchesStrategicMerge:
  - deployment.yaml
```

Patches can be used to apply different customizations to Resources. Kustomize supports different patching mechanisms through `patchesStrategicMerge` and `patchesJson6902`. `patchesStrategicMerge` is a list of file paths.

We can generate the manifests and apply to the cluster by executing (from the base folder):

```shell
kustomize build ./overlay/dev | kubectl apply -f -
```

By running this, only 1 pod will be created in the deployment object, instead of what's defined in the `base` because of the customisation we've applied. We can do the same with prod, or any arbitrary number of environments.

!!! success "Exam Tip"

    Practice with upstream Helm charts for applications that interest you. Dig into how `values.yaml` work