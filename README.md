# ArgoCD : A declarative, GitOps continuous delivery tool

![argocdcover (1)](https://github.com/user-attachments/assets/06a07fab-2dd9-40f4-afb0-849440e08b64)


## What Is Argo CD?

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

## Why Argo CD?

Application definitions, configurations, and environments should be declarative and version controlled. Application deployment and lifecycle management should be automated, auditable, and easy to understand.

## Getting Started

### Quick Start

```
kubectl create namespace argocd
```
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Follow our [getting started guide](https://argo-cd.readthedocs.io/en/stable/getting_started/). Further user-oriented [documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/) is provided for additional features. If you are looking to upgrade Argo CD, see the [upgrade guide](https://argo-cd.readthedocs.io/en/stable/operator-manual/upgrading/overview/). Developer oriented [documentation](https://argo-cd.readthedocs.io/en/stable/developer-guide/) is available for people interested in building third-party integrations.

### **Retrieving Password from Kubernetes Secret**

Run the following command to get the password from the Kubernetes secret:
```
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
```
This will return the initial admin password

### **Solution for PowerShell (Windows)**

Try this command:
```
$pass = kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pass))
```
This will decode the password properly.

### Manually Checking the Password

Run these commands:

```
kubectl get secret -n argocd
kubectl get secret/argocd-initial-admin-secret -n argocd
kubectl get secret/argocd-initial-admin-secret -n argocd -o yaml
```
![image](https://github.com/user-attachments/assets/4abfb330-7a86-44c3-9287-6a436fb2a464)

After that we have to decode with base 64:
```
echo -n "Input the encoded password" | base64 -d
```
### Port-Forward accessing the GUI

To port forward the ArgoCD server so you can access the UI locally, use the following command:

```
kubectl port-forward svc/argocd-server -n argocd 8082:443
or
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8082:443
```
After port-forwarding. We can access the ArgoCD access the graphical user interface using any web browser.
```
https://localhost:8082
```
![image](https://github.com/user-attachments/assets/24a94c0a-a61e-434e-8908-4ffb3f26c80b)

# Setting up Application

First, we have to add the git repository from Settings > **Repositories > Connect Repo**

Then we have to provide the name, repository url etc. in order to add the github repository.

![image](https://github.com/user-attachments/assets/50a27255-d4b7-4766-b0b5-c1349c5998a0)

After that we have to create the application. We will find the option by clicking the three dot menu.

![image](https://github.com/user-attachments/assets/24da045e-7aea-4cda-becf-b27780ef2407)

Again we have to fill up the necessary details. We have to be careful when selecting the Sync policy method. Because using automatic for production branch is not recommended. For other branches like dev, test etc. we can select the automatic method. Because it doesn’t push anything in the production branch.

![image](https://github.com/user-attachments/assets/ff863c03-eb72-4164-80ee-14eab03446d2)

This how it’ll look after finishing the application creation process.

![image](https://github.com/user-attachments/assets/00b2dfd7-dd8b-4511-aecd-17504aaf592d)

![image](https://github.com/user-attachments/assets/690212fd-d947-47b9-96e0-29d384406aca)

# Switching the Images

If we want to change the images of the deployments we have to edit the deployment in the Kubernetes. Here we are going to which from nginx:latest to nginx:stable-alpine3.20-perl. 

We can edit the deployment by using this command:

```
kubectl edit deployment/<deployment name>
```
Then we have to change the image name.

![image](https://github.com/user-attachments/assets/2d674a34-0705-4292-8235-bdcbedd042d0)

After saving the changes. If we check the ArgoCD we will see that the new pod is provisioning using the alpine image.

![image](https://github.com/user-attachments/assets/15977103-5f19-4986-ac9f-092ff5c7e149)

But in this method there's a problem. Are see can in the picture above there is a prompt called OutOfSync. If we sync the processes the pod will be rollback to default nginx:latest.

So, we have to make changes in the source code. After changing the code, we have to push the changes to the GitHub repository.

![image](https://github.com/user-attachments/assets/ad56d1af-28e6-4912-8ee3-723edd7910d5)

If check the ArgoCD interface will see that based on the changes which we made in the GitHub repository, the ArgoCD will automatically update the deployment.

![image](https://github.com/user-attachments/assets/8c73386a-0518-44f3-8c8d-9db7b3879c3c)

# Creating a read only role for developer in RBAC for ArgoCD


First get ConfigMap for the ArgoCD RBAC configuration 

```jsx
kubectl get cm argocd-cm -n argocd -o yaml > argocd-cm.yaml
```

Add data properly right alignment of the metadata

```
data:
  accounts.developer: login
```

Apply the argocd-cm.yaml

```
kubectl apply -f argocd-cm.yaml
```

In order to set password get pod name from list

```
kubectl get pods -n argocd
```

Exec into argocd-server pod

```
kubectl exec -it -n argocd <argocd-server-* name> -- bash

```

After that we need to login with admin creds inside pods with service name

```
argocd login --username admin --password <input password> argocd-server
```

Get account list

```
argocd account list
```

To set the nre password of an account 

```
argocd account update-pasaword --account developer --new-password <set password ex: NewPass123!>
```

To enable the readOnly access developers account configure argocd-rbac-cm.yaml

```
kubectl get cm argocd-rbac-cm -n argocd -o yaml > argocd-rbac-cm.yaml
```

Add data properly right alignment of the metadata

```
data:
  policy.default: role:readonly
```

Apply argocd-rbac-cm.yaml

```
kubectl apply -f argocd-rbac-cm.yaml
```

# Accessing the newly created role

We have to again port forward the ArgoCD server and hit server.
This time we have to login with the new profile username and password.

![image](https://github.com/user-attachments/assets/1e8ec898-768d-4ba7-9c5e-5c2211341213)
![image](https://github.com/user-attachments/assets/4cb13906-a4da-4bd7-9b54-9eab3a92c4be)













