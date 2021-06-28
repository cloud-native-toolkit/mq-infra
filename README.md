# mq-infra

This project contains necessary artifacts for deploying queuemanager with high availability enabled on Openshift.

## Table of Contents

* [Introduction](#introduction)
* [Pre-requisites](#pre-requisites)
* [Deploy Queuemanager](#deploy-queuemanager)

## Introduction

This guide provides a walkthrough on how to set up an Queuemanager.  The Github repository is a template containing a Dockerfile and Helm Chart which is used with the [Cloud Native Toolkit](https://cloudnativetoolkit.dev/) to register a Tekton pipeline to build a Queuemanager image and deploy it on a containerized instance of IBM MQ.

This repo contains the below artifacts.


```
.
├── Dockerfile
├── chart
│   └── base
│       ├── Chart.yaml
│       ├── .helmignore
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── _helpers.tpl
│       │   └── qm-template.yaml
│       └── values.yaml
└── config example
```

- `ibm-mqadvanced-server-integration` docker image that comes with CloudPaks. This image can be further customized if we need additional configurations that are part of queuemanager.
- `Helm Charts` - Currently, we are using quickstart template for deploying the queuemanager.
- `Configurations` - Like mentioned earlier, the configurations can be embedded as part of Dockerfile. Alternatively, they can also be injected as configmaps.

## Pre-requisites

- [IBM Catalog Operator](https://www.ibm.com/docs/en/app-connect/11.0.0?topic=iicia-enabling-operator-catalog-cloud-pak-foundational-services-operator)
- [IBM Common Services](https://github.com/IBM/ibm-common-service-operator)
- [IBM MQ Operator](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=integration-using-mq-in-cloud-pak-openshift)
- [Cloudnative toolkit](https://cloudnativetoolkit.dev/overview)

## Deploy Queuemanager

- Clone the repository locally.
```
  git clone https://github.com/cloud-native-toolkit/mq-infra.git
  cd mq-infra
```

- Create a new project.

```
oc sync dev
```

- Run the pipeline.
```
oc pipeline --tekton
```

- Enter git credentials.
  - Use down/up arrow and select `ibm-mq`.
  - Hit Enter to enable image scanning.
  - Open the url to see the pipeline running in the OpenShift Console.

- Verify that Pipeline Run completed successfully.

- To make sure the queuemanager is successfully deployed, run `oc get queuemanager`.

Note: This pipeline demonstrates how to add in the `mqsc` configuration files. Similarly, if you want to configure an `qm.ini`, please create a configMap for the same and inject it under `spec.queueManager` in the `qm-template.yaml` using the below snippet.

```yaml
ini:
- configMap:
    name: {{ .Values.ini.configmap }}
    items:
    - {{ .Values.ini.name }}
```


