# mq-infra

This project contains necessary artifacts for deploying queuemanager on Openshift.

## Table of Contents

* [Introduction](#introduction)
* [Pre-requisites](#pre-requisites)
* [Queuemanager Details](#queuemanager-details)

## Introduction

This guide provides a walkthrough on how to set up an Queuemanager.  The Github repository is a template containing a Dockerfile and Helm Chart which is used with the [Cloud Native Toolkit](https://cloudnativetoolkit.dev/) to register a Tekton pipeline to build a Queuemanager image and deploy it on a containerized instance of IBM MQ.

This repo contains the below artifacts.


```
.
├── Dockerfile
├── README.md
└── chart
    └── base
        ├── Chart.yaml
        ├── config
        │   └── config.mqsc
        ├── security
        │   └── config.mqsc
        ├── templates
        │   ├── NOTES.txt
        │   ├── _helpers.tpl
        │   ├── configmap.yaml
        │   └── qm-template.yaml
        └── values.yaml
```

- `ibm-mqadvanced-server-integration` docker image that comes with CloudPaks. This image can be further customized if we need additional configurations that are part of queuemanager.
- `Helm Charts` - Currently, we are using quickstart template as our base and building additional things on top of it for deploying the queuemanager.
- `Configurations` - Like mentioned earlier, the configurations can be embedded as part of Dockerfile. Alternatively, they can also be injected as configmaps.

## Pre-requisites

- [IBM Catalog Operator](https://www.ibm.com/docs/en/app-connect/11.0.0?topic=iicia-enabling-operator-catalog-cloud-pak-foundational-services-operator)
- [IBM Common Services](https://github.com/IBM/ibm-common-service-operator)
- [IBM MQ Operator](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=integration-using-mq-in-cloud-pak-openshift)

## Queuemanager Details

- Intially, security and native HA are disabled.
- To enable security, set `security` to true in `Values.yaml`.
- To enable high availability, set `ha` to true in `Values.yaml`.

Note: This project demonstrates how to add in the `mqsc` configuration files. Similarly, if you want to configure an `qm.ini`, please create a configMap for the same and inject it under `spec.queueManager` in the `qm-template.yaml` using the below snippet.

```yaml
ini:
- configMap:
    name: {{ .Values.ini.configmap }}
    items:
    - {{ .Values.ini.name }}
```

### Configuration

- Create required queues to store info.
- Create channel to provide necessary communication links.
- For this queuemanager, channel authentication is disabled.

## Enable Security

If you want to enable the queuemanager to use security, we need to set the `security` flag to `true` in `Values.yaml`. By default, it is always `false`.

### Configuration

- Create required queues to store info.
- Except for the channel used by MQ Explorer, all channels with inbound connections are blocked.
- Create channel to provide necessary communication links.
- Allow access to the LDAP channel.
- Privileged user IDs asserted by a client-connection channel are blocked by means of the special value *MQADMIN* by default. We are removing this default rule.
- For authentication, our queuemanage is using LDAP. We define necessary LDAP configurations. Our sample configurations as follows.

  ```
    DEFINE AUTHINFO(USE.LDAP) +
    AUTHTYPE(IDPWLDAP) +
    CONNAME('openldap.openldap(389)') +
    LDAPUSER('cn=admin,dc=ibm,dc=com') LDAPPWD('admin') +
    SECCOMM(NO) +
    USRFIELD('uid') +
    SHORTUSR('uid') +
    BASEDNU('ou=people,dc=ibm,dc=com') +
    AUTHORMD(SEARCHGRP) +
    BASEDNG('ou=groups,dc=ibm,dc=com') +
    GRPFIELD('cn') +
    CLASSGRP('groupOfUniqueNames') +
    FINDGRP('uniqueMember') +
    CHCKCLNT(REQUIRED) +
    REPLACE
    ALTER QMGR CONNAUTH(USE.LDAP)
  ```
- Enable TLS protocol security. We can specify the cryptographic algorithms that are used by the TLS protocol by supplying a CipherSpec as part of the channel definition along with the authenticated user information.
- This privileged user will be allowed to access the queue manager and interact with it.

## Enable Native HA

If you want to enable the queuemanager to use native high avaibility capability, we need to set the `ha` flag to `true` in `Values.yaml`. By default, it is always `false`.

- A Native HA configuration provides a highly available queue manager where the recoverable MQ data (for example, the messages)  are replicated across multiple sets of storage, preventing loss from storage failures.
- This is suitable for use with cloud block storage.
