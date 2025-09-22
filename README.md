# Overview

This is the gitops repo for the metrics-collector-py project (https://github.com/srivathsashreyas/metrics-collector-py). It contains packages the relevant components in the project as a helm chart. You can use this for a local deployment as well. But if you're planning to experiment, consider using the Dockerfile and manifests provided in the main repo to build and deploy the components. 

Note: The image tags provided in the values.yaml file will be updated 
based on changes in the main repo.

# ArgoCD Setup
1. You can install argocd in your cluster using the following commands (from the official docs -> https://argo-cd.readthedocs.io/en/stable/getting_started/):
    -  Create the namespace for argocd resources -`kubectl create namespace argocd`
    - Install argocd - `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` 
    - Install argocd CLI (macOS only, you may need to refer the docs for other OS) - `brew install argocd`
    - Expose the argocd server by forwarding traffic directed to port 8080 on your local machine to port 443 on the argocd-server service: `kubectl port-forward svc/argocd-server -n argocd 8080:443`
    - Retrieve the initial password to login to argo cd ui - `argocd admin initial-password -n argocd`
    - Login to the argocd UI at `localhost:8080` using the username `admin` and password retrieved from the previous step
    - Apply the application manifest provided in this repo - `kubectl apply -f argocd/application.yaml` to sync the gitops repo with the cluster. You should be able to view the resources being created and their status (healthy, error, etc) in the argocd UI.
Note: These steps have been tested on a k3s cluster running on a colima VM on macOS. There may be slight differences based on the setup you are using.