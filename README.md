# Node-RED Tekton Pipeline
This repo contains example Tekton scripts for deploying a [node-RED](https://nodered.org/docs/) pipeline on Kubernetes or OpenShift Container Platform.

The `pipeline` folder contains all the Tekton resources needed to create a pipline for a Node-RED deployment.

The `k8s` folder contains example deployment resources and Dockerfile that should be included in your node-RED project repo, so that the pipeline tasks can use these to build the project.

## Overview
There are a number of ways of creating a CI/CD pipeline for node-RED. This strategy will inject the flows.json data into the container image at build time and then push this image to an image repo. This provides you with version control and the ability to roll back deployments.


This Tekton strategy contains 4 tasks:
- clone-repo (cluster task)
- create-image
- kustomize-build
- deploy-app

The Dockerfile assumes you are saving your flows.json data to the root of the project folder. You should not rename or move this file.

## Step 1 Prerequisites
- Change app name and namespace values in the `k8s` folder: `deployment.yaml`, `route.yaml` and `service.yaml`
- Change image repo in `deployment.yaml`
- Change the app name and namespace values in all resources from the `pipeline` folder


## Step 2 Prerequisites: OCP / Kubernetes 
- Add your Github secret:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: node-red-examle-git-secret
data:
  ssh-privatekey: >-
  <your private ssh key>
type: kubernetes.io/ssh-auth

```

- Add your Image Repo secret, this example uses ICR (for pushing images):

```bash
oc -n <my_namespace> create secret docker-registry icr-registry-token \
--docker-server=uk.icr.io \
--docker-username=iamapikey \
--docker-password=<ibm_cloud_iam_api_key> \
--docker-email=<docker_email>
```

- Add secrets to the pipeline service account:
```bash
oc patch -n <my_namespace> serviceaccount/pipeline --type='json' -p='[{"op":"add","path":"/secrets/-","value":{"name":"node-red-example-git-secret"}}]'

oc patch -n <my_namespace> serviceaccount/pipeline --type='json' -p='[{"op":"add","path":"/secrets/-","value":{"name":"icr-registry-token"}}]'
```

- Copy Image Repo secret from default namespace (for pulling images):

```bash
oc get secret all-icr-io -n default -o yaml \
| sed 's/namespace: default/namespace: <my_namespace>/g' \
| kubectl -n <my_namespace> create -f -
```

- Add Image Repo secret to default service account (for pulling images):
```bash
oc patch -n <my_namespace> serviceaccount/default --type='json' -p='[{"op":"add","path":"/imagePullSecrets/-","value":{"name":"all-icr-io"}}]'
```

## Deploying pipelines
From the `pipeline` folder:
- Deploy all resources within `tasks` and `trigger-bindings` folders first
- Deploy resources inside `trigger-templates`, `pipelines` and `event-listeners` folders

## Webhooks setup

- Make sure that the event listener services are exposed:
```bash
oc expose svc <el-service>
```
- Get the route for event listener and add it to the webhooks section on github