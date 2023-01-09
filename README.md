# **Flux CD Assignment**
---
 
 ## **TASK : GitOps with Flux - Practice** 
 
- Install and configure flux with GitHub repo/branch
- fork this repo https://github.com/prakhar1989/docker-curriculum/tree/master/static-site  
- Build a docker image and push it to dockerhub
- write a sample helm template to deploy the image that we built above with a single pod
- Release application after the PR is merged and that version of docker image is pushed to registry
- Deploy sample application (helm)  FluxCD
- Deploy revision based on docker image push (Public docker repo)  
- Deployment and Release Management
  - Helm release management
  - To explore on integrating third party UI solutions with Flux
- Good to have
  - Flagger(istio) integration to achieve (blue-green/canary)Progressive delivery
  - Manage flux through declarative manifest(yaml)

---

### **SOLUTION :**

#### Flux configured here works as below :
  - First it bootstrap the github repo. if repo is already available it will reconcile that else it will create one with default component's(and extra components if given through cli) menifests on branch given in cli argument. And will deploy those component to cluster.
  - Controllers that are required for the task :
    ```
    default : source-controller,kustomize-controller,helm-controller,notification-controller
    
    extra : image-reflector-controller,image-automation-controller
    ```
  - Resource that are created :
    ```
    gitrepository/flux-system --> To sync cluster with flux_assignment repo
    helmrepository/my-helm-repo --> To add custom helm repo to cluster
    helmchart/flux-system-flux-static-site --> To configure chart on the cluster
    imagerepository/static-site-image-repo --> To define registry that need to be scanned on defined intervals
    imagepolicy/static-site-image-update-policy  --> This will keep scanning registry for matching conditions such as tags etc.
    imageupdateautomation/static-site-image-update --> This will update and push the helm release manifest to flux_assignment repo with if there is any image found with matching conditions as per image policy  
    helmrelease/flux-static-site --> To release helm chart (Custom values can be defined here)
    kustomization/flux-system --> This will be created and managed by flux automatically.

    FOR deploying weaveworks UI solution :
     - helmrepository/ww-gitops
     - helmchart/flux-system-ww-gitops
     - helmrelease/ww-gitops
    ```
  
#### Repos and Image Registry that are used in this assignment :
 - https://github.com/bhagyeshsoni98/flux-static-site for Docker image.
   - I have created github action workflow that will be triggered on pushing the changes to main branch. It will build and push docer image to Registry(https://hub.docker.com/repository/docker/bhagyesh19/static-site/general) on docker hub.
   - Github Action Workflow Code :
      ```bash
      name: ci

      on:
        push:
          branches:
            - 'main'

      jobs:
        docker:
          runs-on: ubuntu-latest
          steps:
          
          - uses: actions/checkout@v2
            name: Check out code

          - name: Get docker image version
            id: vars
            run: echo "version=$(cat docker_image_version)" >> $GITHUB_OUTPUT
            
          - uses: mr-smithers-excellent/docker-build-push@v5
            name: Build & push Docker image
            with:
              image: bhagyesh19/static-site
              tags: ${{ steps.vars.outputs.version }}
              registry: docker.io
              username: ${{ secrets.DOCKER_USERNAME }}
              password: ${{ secrets.DOCKER_PASSWORD }}
      ```
    - https://hub.docker.com/repository/docker/bhagyesh19/static-site/general Docker Registry for hosting custom image
    - https://github.com/bhagyeshsoni98/flux-custom-helm-chart to host custom helm package on Github Pages


#### Steps to configure flux :

1. Bootstrap github repo :
   
    ![flux_bootstrap](././media/flux_bootstrap.png)
    
    ![pod_deployed_after_bootstrap](././media/pod_deployed_after_bootstrap.png)

    ![flux_resources](././media/flux_resources.png)

   - Create GITHUB_USER and GITHUB_TOKEN(github personal token)
   - Fire below command to bootstrap the repo
      ```bash
       flux bootstrap github --owner=$GITHUB_USER --repository=flux_assignment --branch=main --path=app-cluster --components-extra 'image-reflector-controller,image-automation-controller' --personal --interval=30s --read-write-key
      ```
      > Flags :
      ```bash
      --interval duration       sync interval (default 1m0s)
      --owner string            GitHub user or organization name
      --path safeRelativePath   path relative to the repository root, when specified the cluster sync will be scoped to this path
      --personal                if true, the owner is assumed to be a GitHub user; otherwise an org
      --read-write-key          if true, the deploy key is configured with read/write permissions
      --repository string       GitHub repository name
      --components-extra strings               list of components in addition to those supplied or defaulted, accepts values such as 'image-reflector-controller,image-automation-controller'
      --branch string                          Git branch (default "main")
      ```
    - This will sync all the yaml that are present at app-cluster path and deploy the components to the k8s cluster.
    - Extra components image-reflector-controller and image-automation-controller are required on cluster to keep scaning image registry at defined intervals and to auto update image in the yaml on github which will eventually update the cluster on next sync.
    - read-write-key is required if you want to auto update your manifest through any flux controller(in this case : image-automation-controller)
    - personal arg fetches personal token from GITHUB_TOKEN env variable.


2. Updating repo https://github.com/bhagyeshsoni98/flux-static-site version which build and push docker image to docker hub.
    ![static_site_repo_commit](./media/static_site_repo_commit.png)
    ![static_site_repo_workflow](./media/static_site_repo_workflow.png)
    ![docker_hub_latest_image](./media/docker_hub_latest_image.png)

3. After image has been pushed to docker hub, imagepolicy resource will scan for latest image in registry. Based on latest image scan, imageupdateautomation will update helm_release.yaml with latest image tag and push it to github repo flux_assignment. Then gitrepository resource sync with flux_assignment repo and create new helm revision with update image.
    ![flux_image_policy_and_update](./media/flux_image_policy_and_update.png)
    ![github_helm_release](./media/github_helm_release.png)
    ![helm_revision](./media/helm_revision.png)
    ![new_pod_created](./media/new_pod_created.png)

4. Port Forwarded svc and verified that static site is accesible through localhost.
    ![port_forwarded_svc](./media/port_forwarded_svc.png)
    ![port_forward](./media/port_forward.png)
    ![localhost](./media/localhost.png)

5. Port forwarded service of weavework UI solution for Flux to manage and view flux resources through UI.
    ![ui_port_forwarded](./media/ui_port_forwarded.png)
    ![ui_1](./media/ui_1.png)
    ![ui_2](./media/ui_2.png)
    ![ui_3](./media/ui_3.png)
    ![ui_4](./media/ui_4.png)
    ![ui_5](./media/ui_5.png)

6. Fire `flux uninstall` command to delete all the flux resources from cluster created under flux-system namespace.
    ![flux_uninstall](./media/flux_uninstall.png)
    ![flux_system_ns_removed](./media/flux_system_ns_removed.png)