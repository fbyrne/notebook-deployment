# notebook-deployment

This project manages the deployment of the notebook application to a kubernetes cluster.  

## Application Design

![Application Design](./notebook-design.png)

The application has the following key points:

1. Identity Authoriation Management (IAM) using an OAuth 2.0 service, in this case keycloak.
1. An Rest API implmented using Spring Boot, Security, WebFlux.
1. A MongoDB store to persist user notes.
1. An AMQP compatible messaging service, in this base RabbitMQ.
   - An exchange `notes` with bindings to the following queues is used:
     - `notes-event-created`
     - `notes-event-updated`
     - `notes-event-deleted`
1. A backend email service to listen and notify users of updates.
1. A frontend SPWA implemented using Reactjs

Please note, the front end is a work in progress and currently only provides authentication and displays the users JWT token.

The service code is located in the following repositories:
- [notebook/note-service](https://github.com/fbyrne/notebook-note-service)
- [notebook/email-service](https://github.com/fbyrne/notebook-email-service)
- [notebook/note-app](https://github.com/fbyrne/notebook-notes-app)


## Deployment Design

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
$ kubectl create namespace notebook-dev
$ skaffold run
```

To deploy for staging:

```shell script
$ kubectl create namespace notebook-staging
$ skaffold run --profile=staging
```

To deploy for production:

```shell script
$ kubectl create namespace notebook-prod
$ skaffold run --profile=prod
```

## Deleting a deployment
```shell script
$ skaffold delete
$ skaffold delete --profile=staging
$ skaffold delete --profile=prod
```

## CI/CD deployment

A sample CI/CD deployment would be broken up into two tools:

1. A CI tool to build and deploy the application images, such as, [Jenkins](http://www.jenkins.io), Google Cloud Build, Travis or Circle CI.
1. A CD tool to deploy changes to the kubernetes deployment files, such as, [ArgoCD](https://argoproj.github.io/argo-cd/)

![CI/CD Example](./notebook-sdlc.png)

### Building on CI

The CI pipeline should use the usual build, test and analyse using the language/framework specific tools.
For deploying the image `skaffold` can be used to build the container images and deploy them to the container registry using deployment specific tags.

```shell script
$ cd ./notebook-note-service; skaffold build --profile=staging
```

This will produce a container image which is tagged `notebook/note-service:staging-<git-tag>` and `notebook/note-service:staging-<commit-sha>`.

The same example works for the production profile, which will produce a container image which is tagged `notebook/note-service:production-<git-tag>` and `notebook/note-service:production-<commit-sha>`.

### Deploying with ArgoCD

ArgoCD can be setup to listen to triggers from the git source repository.

For `staging` continuous deployment, the argocd pipeline can be triggered on push events for the `staging` branch or tag.

For `production` continuous deployment, the ArgoCD pipeline can be triggered on push events for the `production` branch or tag.
