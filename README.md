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
```
Forward the ArgoCD UI to localhost:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
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
- Repo URL: `https://github.com/himanshukr8090/DevOps_Project_Deploying_Wordpress_Bloging_Site_on_AWS`
- Path: `/`
- Namespace: default
- Cluster URL: https://kubernetes.default.svc
#### Click Create and then Sync the application to deploy.  

#### 🚀 ArgoCD Dashboard Images:
![Screenshot 2025-05-17 105010](https://github.com/user-attachments/assets/f13637a3-8ea3-4b53-af9d-836f814a3416)

![Screenshot 2025-05-17 105115](https://github.com/user-attachments/assets/2e576601-d3ce-4dae-8118-ea840dd37e9d)


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
```

### 7. 🌍 Access the WordPress Blog
- Use the external IP or DNS from the LoadBalancer service to open the WordPress installation page:

- Example:
`http://afac9b7b1acb04c109118ebfee30527a-1772024785.ap-south-1.elb.amazonaws.com`

#### Follow the on-screen steps to configure WordPress with your MySQL backend.

- 1 ![Screenshot 2025-05-17 105809](https://github.com/user-attachments/assets/9ce10b23-81dd-4d61-9c88-c385159ac22f)
- 2 ![Screenshot 2025-05-17 105825](https://github.com/user-attachments/assets/300a72da-80b9-4059-b412-533bd05b8ba9)
- 3 ![Screenshot 2025-05-17 111026](https://github.com/user-attachments/assets/96ac7978-714e-4edc-a79e-74de583b9f9a)
- 4 ![Screenshot 2025-05-17 111132](https://github.com/user-attachments/assets/75df3787-462e-4c14-bd37-8bc955e5b33e)
- 5 ![Screenshot 2025-05-17 111950](https://github.com/user-attachments/assets/fde463c6-a6df-4a07-9869-9739fa587b90)
- 6 ![Screenshot 2025-05-17 112002](https://github.com/user-attachments/assets/289deaa8-9d6d-442e-9970-a0795850682d)
- 7 ![Screenshot 2025-05-17 112441](https://github.com/user-attachments/assets/be20fcbb-8a4a-4fb3-b048-e8337f11c30d)
- 8 ![Screenshot 2025-05-17 112519](https://github.com/user-attachments/assets/7e0aa8f3-239e-4023-afdf-1586dca81b26)


### 8. 🧹 Cleanup (Optional)
####To delete the EKS cluster and all associated resources:

```bash
eksctl delete cluster --name my-cluster --region ap-south-1
```
## This Project is Created by `Himanshu Kumar Singh`
