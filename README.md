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

# Deployment using Microshift
1. Install Microshift by first installing RedHat Openshift Local. Then configure the microshift preset before setup (refer https://crc.dev/docs/using/):
    - `crc delete # Remove previous cluster (if present)`
    - `crc config set preset microshift # Configure to use microshift preset (do not forget this because the openshift preset is default)`
    - `crc setup # Initialize cluster`
    - `crc start # Start the cluster`
2. Ensure that you have also installed the oc cli.
3. Install argocd based on the steps provided above. If the argocd-redis pod isn't running:
    - `kubectl get deployment argocd-redis -n argocd -o yaml` -> Retrieve the argocd redis deplpoyment manifest. Look for an error message (at the bottom) containing info in the form: 'pods "argocd-redis-6c86cd7988-" is forbidden: unable to validate against any security context constraint: [provider "anyuid": Forbidden: not usable by user or serviceaccount, provider restricted-v2: .initContainers[0].runAsUser: Invalid value: 999: must be in the ranges: [1000130000, 1000139999]'. 
    Modify the runAsUser value in the argocd/argocd-redis.yaml manifest provided in this repo to match the range in the error message (in this case 1000130000-1000139999)
    - Delete the argocd-redis deployment - `kubectl delete deployment argocd-redis -n argocd`
    - Apply the modified manifest using `kubectl apply -f argocd/argocd-redis.yaml -n argocd`
    - Restart all other argocd deployments and stateful sets - `kubectl rollout restart deploy/<deployment name> -n argocd` and `kubectl rollout restart statefulset deploy/<deployment name> -n argocd`
    - Port forward the argocd-server service as mentioned in the previous section and login to the UI
