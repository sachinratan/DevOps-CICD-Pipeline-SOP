## This guide will help you to install the helm binary and create the helm chart repo for application deployment.

#### Install helm - [reference](https://helm.sh/docs/intro/install/)

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

#### Create helm chart repository
```
$  helm create jnks-k8s-app -n <namespace>
```

#### Directory structure for helm chart repository

```
For example, 'helm create foo' will create a directory structure that looks
something like this:

    foo/
    ├── .helmignore   # Contains patterns to ignore when packaging Helm charts.
    ├── Chart.yaml    # Information about your chart
    ├── values.yaml   # The default values for your templates
    ├── charts/       # Charts that this chart depends on
    └── templates/    # The template files
        └── tests/    # The test files

```
