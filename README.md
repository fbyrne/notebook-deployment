# notebook-deployment

This project manages the deployment of the notebook application to a kubernetes cluster.  

It is designed to use [kustomize](https://kubernetes-sigs.github.io/kustomize/) to deploy multiple configuration from the same base kubernetes resource manifests.

## Installing [kustomize](https://kubernetes-sigs.github.io/kustomize/)

Installation instructions can be found @ https://kubernetes-sigs.github.io/kustomize/installation.

```shell script
curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
sudo install kustomize /usr/local/bin
```

## Structure

The base resources are contained within the '/base' directory.  The base kustomize file defines which resources to include and any common customization to apply to the resources.

The overlays folder contains a folder for each deployment types that are defined.

## Deploying to a kubernetes cluster using `kubectl`

Using kustomize many different types of deployments (dev, staging, prod) can be managed for a common set of kubernetes resources.  The deployments are deployed to the cluster using `kubectl` as follows:

```shell script
kustomize build ./overlays/dev | kubectl apply -n notebook-dev -f -
```

### Deploying to a kubernetes cluster using [Skaffold](https://skaffold.dev/)

Skaffold handles the workflow for building, pushing and deploying your application, allowing you to focus on what matters most: writing code.

To deploy for development purposes, and watching for changes:

```shell script
$ skaffold dev
```

To deploy for development purposes:

```shell script
$ skaffold deploy
```

To deploy for staging:

```shell script
$ skaffold deploy --profile=staging
```