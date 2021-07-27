# Tekton Pipeline and Cluster Configuration

This steps should be followed for setting up a new CI/CD pipeline.

## Requirements

1. Account at [quay.io](https://quay.io)

   1. Repository `next-js-app`
   1. Robot account with `write` rights to the repository 🤖

1. [Github](https://github.com)

   1. Code repository `end-to-end-assignment-next-js`
   1. GitOps repository `end-to-end-assignment-next-js-gitops`
   1. Personal access token with full control of private repositories

   If you change repository names, be sure that you change it in all `yaml` files to match your needs.

## Tasks

Some Tasks are required on a cluster level, others on a project level

### Cluster Tasks

- [Git Clone](https://hub.tekton.dev/tekton/task/git-clone)
- [Buildah](https://hub.tekton.dev/tekton/task/buildah)

### Project Tasks

- [NPM](https://hub.tekton.dev/tekton/task/npm)
- kostumize-build

  ```bash
  # From the root folder run
  oc apply -f ./k8s/tasks/kustomize-build.yaml
  ```

- test-deploy

  ```bash
  # From the root folder run
  oc apply -f ./k8s/tasks/test-depoly.yaml
  ```

- commit-manifest

  ```bash
  # From the root folder run
  oc apply -f ./k8s/config/config-map.yaml
  ```

## Get the cluster configured

1. Set up the **secret** for the `build-bot` service account  
   This will push the build image to `quay.io`

   1. Duplicate and rename the `next-js-secret.yaml.example` to `next-js-secret.yaml`
   1. Insert your API key at `line 6: <YOUR API KEY>`
   1. Apply the secret to the cluster

      ```bash
      # From the root folder run
      oc apply -f ./k8s/config/next-js-secret.yaml
      ```

1. Set up the `build-bot` **service account** for pipeline `trigger template` and `buildah image build` and push

   ```bash
   # From the root folder run
   oc apply -f ./k8s/config/build-bot-service-account.yaml
   ```

1. Create your **git credentials**
   Needed for pushing the `manifest.yaml` to the GitOps repository

   1. Duplicate and rename the `git-credentials.yaml.example` to `git-credentials.yaml`
   1. Insert your github username at `line 7: <YOUR GIT-HUB USERNAME>`
   1. Insert your github access token at `line 8: <YOUR GIT-HUB PERSONAL ACCESS TOKEN>`
   1. Apply the secret to the cluster

      ```bash
      # From the root folder run
      oc apply -f ./k8s/config/git-credentials.yaml
      ```

1. Set up the **config map** which holds information to push to the GitOps repository  
   Configure the following values to your needs:

   - `line 9: repo: <YOUR GIT-OPS REPO>`
   - `line 10: org: <USERNAME OR ORGANISATION>`

   After this apply the changes

   ```bash
   # From the root folder run
   oc apply -f ./k8s/config/config-map.yaml
   ```

## Pipeline setup

1. Bringing the pipeline in place

   ```bash
   # From the root folder run
   oc apply -f ./k8s/pipeline/pipeline.yaml
   ```

1. Afterwards create the pipeline run

   ```bash
   # From the root folder run
   oc create -f ./k8s/pipeline/pipeline-run.yaml
   ```

   This should trigger the first run for the complete pipeline

## Trigger setup

To get the pipeline connected to the repository, a trigger is needed.

1. A `service account` is needed for the trigger to run `pipeline runs` and grant access to further `resources`

   ```bash
   # From the root folder run
   oc apply -f ./k8s/trigger/trigger-service-account.yaml
   ```

   This creates a `service account`, the `role based access controls` and the `binding` to connect them.

1. The route to access the trigger from outside the cluster

   ```bash
   # From the root folder run
   oc apply -f ./k8s/trigger/trigger-route.yaml
   ```

1. The trigger template. References the `pipeline`, `workspace` and holds the parameter for the pipeline e.g. upload target for the resulting image

   ```bash
   # From the root folder run
   oc apply -f ./k8s/trigger/trigger-template.yaml
   ```

1. An event listener, which references the `service` accounts for the needed rights

   ```bash
   # From the root folder run
   oc apply -f ./k8s/trigger/trigger-event-listener.yaml
   ```

1. The `Trigger Binding` specifies the fields in the event payload from which the data is extracted

   ```bash
   # From the root folder run
   oc apply -f ./k8s/trigger/trigger-binding.yaml
   ```

### Webhook

For setting up the webhook the url of the trigger is needed.
Retrieve it with the following command:

```bash
oc get routes | grep vcs-trigger
# Expected output similar to
# vcs-trigger-fabian-next-js.ibmcloud-roks-z9a7g-6ccd7f378ae819553d37d5f2ee142bd6-0000.ams03.containers.appdomain.cloud
```

For the full url add `http://` and add this in your repository's webhook as payload.  
Set the **content type** to `application/json`.

## Manually trigger the pipeline

```bash
oc get routes | grep vcs-trigger
# Expected output similar to
# vcs-trigger-fabian-next-js.ibmcloud-roks-z9a7g-6ccd7f378ae819553d37d5f2ee142bd6-0000.ams03.containers.appdomain.cloud
```
