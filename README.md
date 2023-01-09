# Flux CD Assignment
---
 
 ## TASK : GitOps with Flux - Practice 
 
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
