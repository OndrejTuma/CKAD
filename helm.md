# Helm

A package/release manager for k8s. 

Makes easier deploying, updating, deleting your applications.

## Install Helm

https://helm.sh/docs/intro/install/

## Concepts

### Templates

Holds definition files of your application

### values.yaml

Provides variables for templates

### Chart

Includes:

- templates
- values.yaml
- Chart.yaml
  - information about the chart itself (version, name, description etc)
  - like package.json

> You can either create your own chart or 
> download existing one in artifacthub.io or other repositories (bitnami)

#### Search the hub

```shell
helm search hub <keyword>
```

#### Search in other repository

1. add repository to your local helm setup
   1. `helm repo add bitnami https://charts.bitnami.com/bitnami`
2. search the repo
   1. `helm repo list`
   2. `helm search repo <keyword>`

## Commands

#### Install 

Install your app. Each installation has a release name.

```shell
helm install [release-name] [chart-name] [flags]
```

#### List

List running charts

```shell
helm list
```

#### Pull

Pull chart to local helm

```shell
helm pull [chart URL | repo/chartname] [flags] 
```

Flags:
- --untar - downloads unzipped version of the chart

#### Repo

```shell
helm repo [command]
```

Commands:

- add
- index
- list
- remove
- update

#### Rollback 

```shell
helm rollback <release-name> [flags]
```

#### Uninstall 

```shell
helm uninstall <release-name> [flags]
```

#### Upgrade 

```shell
helm upgrade [release-name] [chart-name] [flags]
```