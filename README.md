# 🚀 DevOps Project: Deploying a WordPress Blogging Site on AWS using EKS and ArgoCD

This project demonstrates how to deploy a WordPress blogging platform on AWS using **Amazon EKS (Elastic Kubernetes Service)**, **ArgoCD** for GitOps-based CD, and Kubernetes resources managed via `kubectl`.

## 📋 Prerequisites

- AWS CLI configured with necessary IAM permissions  
- `eksctl`, `kubectl`, and `argocd` CLI tools installed  
- A GitHub repository containing Kubernetes manifests  

---

## 🛠️ Step-by-Step Implementation

### 1. ✅ Create EKS Cluster

```bash
eksctl create cluster \
  --name my-cluster \
  --node-type t3.medium \
  --region ap-south-1 \
  --nodes 1 \
  --managed
NOTE: we can create cluster using KIND cluster also.
```

### 2. 🚀 Install ArgoCD on the Cluster
Create a namespace and install ArgoCD:
``` bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Retrieve the admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd
OR
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```
Forward the ArgoCD UI to localhost:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
or
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address=0.0.0.0
```
![Screenshot 2025-05-17 123503](https://github.com/user-attachments/assets/23cd4539-4053-4c78-b706-4dc9f644586b)

#### Login to ArgoCD
![Screenshot 2025-05-17 104549](https://github.com/user-attachments/assets/e580319b-ed6e-4a9f-90fe-d6485be83ed0)

### 3. 🌐 Update GitHub and push
#### Push Files to github: 
```
$ ls
README.md  mysql_deployment.yml  mysql_secret.yml  wordpress_deployment.yml

git add .
git commit -m "Deploying Wordpress Bloging Sites with Mysql on AWS"
git push
```

### 4. 📦 Create ArgoCD Application
#### From the ArgoCD web UI:
- Click `New App`
- App Name: `wordpressapp`
- Project: `default`
- Sync Policy: `Manual` (or `Auto` if preferred)
- Repo URL: `https://github.com/juhisinha422/DevOps_Project_Deploying_Wordpress_Bloging_Site_on_AWS`
- Path: `/`
- Namespace: default
- Cluster URL: https://kubernetes.default.svc
#### Click Create and then Sync the application to deploy.  

#### 🚀 ArgoCD Dashboard Images:
![Screenshot 2025-08-24 105010](https://github.com/user-attachments/assets/05d28e46-c507-4f84-8e6b-0acb0aa6781a)

![Screenshot 2025-08-24 105115](https://github.com/user-attachments/assets/971d550c-b934-45d8-b0a2-18d56cb51d8d)


### 5. 🧱 Kubernetes Manifests Overview
 
`mysql_secret.yml` – stores MySQL username and password securely  
`mysql_deployment.yml` – defines MySQL pod and service  
`wordpress_deployment.yml` – defines WordPress pod and service

### 6. 🔍 Verify Kubernetes Resources
#### Check pods:
```bash
kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mysqldb1-76d69b9646-ctdfm    1/1     Running   0          74s
wpressapp-68b68b65bb-l2hdn   1/1     Running   0          74s
```
#### Check services:
```bash
$ kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP                                                                PORT(S)        AGE
kubernetes   ClusterIP      10.100.0.1       <none>                                                                     443/TCP        2m23s
mysqldb1     ClusterIP      10.100.167.138   <none>                                                                     3306/TCP       47s
wpressapp    LoadBalancer   10.100.39.135    afac9b7b1acb04c109118ebfee30527a-1772024785.ap-south-1.elb.amazonaws.com   80:31034/TCP   47s

OR (if using KIND cluster)

$ kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          155m
mysqldb1           ClusterIP   10.96.220.127   <none>        3306/TCP         55m
wpressapp          NodePort    10.96.49.173    <none>        80:30328/TCP     80s

```

### 7. 🌍 Access the WordPress Blog
- Use the external IP or DNS from the LoadBalancer service to open the WordPress installation page:

- Example:
`http://afac9b7b1acb04c109118ebfee30527a-1772024785.ap-south-1.elb.amazonaws.com`

NOTE: We can also expose using NodePort IP. (In wordpress deployment yml give type as NodePort)

 kubectl port-forward svc/wpressapp 30328:80 --address=0.0.0.0 (if using KIND and NodePort IP. (kubectl get svc wordpress)

#### Follow the on-screen steps to configure WordPress with your MySQL backend.

NOTE:Give database host name (10.96.220.127) --kubectl get svc (mysql)
- 1 ![Screenshot 2025-08-24 105809](https://github.com/user-attachments/assets/c164707f-87c4-4b85-9c0d-9bfbb85f15fd)
- 2 ![Screenshot 2025-08-24 105825](https://github.com/user-attachments/assets/5e3aecac-a097-4b9b-a6a1-e6bd37fad15e)
- 3 ![Screenshot 2025-08-24 111026](https://github.com/user-attachments/assets/ecc1dcd5-e451-4522-8dae-271cb1bfa844)
- 4 ![Screenshot 2025-08-24 111132](https://github.com/user-attachments/assets/6cebdc93-b49b-4ca1-83b2-623a06743521)
- 5 ![Screenshot 2025-08-24 111950](https://github.com/user-attachments/assets/94a1cab2-23cb-4119-b36e-c7bd898930ac)
- 6 ![Screenshot 2025-08-24 112002](https://github.com/user-attachments/assets/289deaa8-9d6d-442e-9970-a0795850682d)
- 7 ![Screenshot 2025-08-24 112441](https://github.com/user-attachments/assets/0bb9c3de-1917-4f4a-b2d3-0277bb9e49dc)
- 8 ![Screenshot 2025-08-24 112519](https://github.com/user-attachments/assets/91720f85-9772-49a3-840f-3009ff0e0d07)


### 8. 🧹 Cleanup (Optional)
####To delete the EKS cluster and all associated resources:

```bash
eksctl delete cluster --name my-cluster --region ap-south-1
OR
kind delete cluster --name <cluster-name> (if using KIND Cluster)
```
## This Project is Created by `Juhi Sinha`
