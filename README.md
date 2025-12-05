# 🛡️ Secure EKS Cluster with Enforced Policies (Gatekeeper), Private Networking, and VPC Flow Logs

### **A Cloud Security Engineer Lab Deploying Enterprise-Grade Controls with Terraform**

<img width="839" height="93" alt="$$$$$" src="https://github.com/user-attachments/assets/8c4542a4-1d7c-44f2-89a9-413535b43979" />


---

## 📌 **Overview**

This project deploys a **fully private, production-grade Amazon EKS cluster** using **Infrastructure as Code (Terraform)**.
It implements **enterprise cloud security controls** such as:

* A **custom VPC** with isolated public & private subnets
* **NAT Gateway** for outbound-only private node traffic
* **Encrypted VPC Flow Logs** delivered to S3
* **EKS cluster with OIDC enabled**
* **Gatekeeper (Open Policy Agent)** installed via Helm
* **Kubernetes admission policies** that *block privileged containers*
* **Validation test pods demonstrating policy enforcement**

This project showcases hands-on experience in:

✔ Cloud Security
✔ Kubernetes Security
✔ Network Visibility
✔ Terraform IaC
✔ Policy-as-Code
✔ AWS EKS + Gatekeeper governance

---

## 🧱 **Architecture Diagram**

```
                   ┌──────────────────────────────┐
                   │          AWS Region          │
                   └──────────────────────────────┘
                              │
     ┌────────────────────────────────────────────────────┐
     │                     Secure VPC                     │
     │   (10.0.0.0/16, DNS enabled, Flow Logs enabled)    │
     └────────────────────────────────────────────────────┘
       │                      │                        │
       │                      │                        │
 Public Subnets         Private Subnets            S3 Bucket
  (2 AZs)                   (2 AZs)            (Encrypted VPC Logs)
       │                      │
       │                      └──────────┐
   Internet GW                           │
                                         ▼
                                 ┌────────────────┐
                                 │  EKS Cluster   │
                                 │  (Private API) │
                                 └────────────────┘
                                          │
                                          ▼
                               ┌────────────────────┐
                               │ Gatekeeper / OPA   │
                               │  Policy Engine     │
                               └────────────────────┘
                                          │
                                          ▼
                             Block Privileged Pods ❌
```

---

## 🚀 **Features Implemented**

### 🔹 **1. Secure VPC**

* Custom CIDR block: `10.0.0.0/16`
* Two public + two private subnets
* NAT gateway for private subnets
* Internet gateway for public routing

<img width="1669" height="871" alt="Screenshot 2025-12-05 at 09 34 06" src="https://github.com/user-attachments/assets/d5846e25-ef33-4bd4-b376-a0f0fc7b78de" />

---

<img width="1673" height="692" alt="Screenshot 2025-12-05 at 09 35 45" src="https://github.com/user-attachments/assets/c8873d7e-0b0c-4f96-992e-9f669400e0eb" />

---

<img width="1673" height="574" alt="Screenshot 2025-12-05 at 09 35 25" src="https://github.com/user-attachments/assets/27760474-cfce-4769-a067-5f19ba4ab2e7" />

### 🔹 **2. VPC Flow Logs → S3 Bucket**

* Flow logs created via Terraform
* Logs stored in a dedicated S3 bucket
* Access controlled by AWS IAM
* Verified working via network traffic generation

<img width="1600" height="772" alt="Screenshot 2025-12-05 at 14 16 49" src="https://github.com/user-attachments/assets/a06b68f7-241d-4003-afa4-63ad90c1b41d" />

---

### 🔹 **3. EKS Cluster (Terraform Managed)**

* EKS version: **1.30**
* Control plane private endpoint enabled
* Worker nodes deployed across two AZs
* OIDC provider automatically created

<img width="1669" height="871" alt="Screenshot 2025-12-05 at 09 41 05" src="https://github.com/user-attachments/assets/d0a42144-fb41-4d02-b5cd-9c9947925878" />

---

<img width="1669" height="871" alt="Screenshot 2025-12-05 at 09 46 00" src="https://github.com/user-attachments/assets/430cff65-d734-49c8-aa51-79c6b33e1d88" />

---

### 🔹 **4. Gatekeeper Policy Enforcement**

Installed via Helm:

```bash
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper gatekeeper/gatekeeper --namespace gatekeeper-system
```
<img width="831" height="799" alt="Screenshot 2025-12-04 at 16 04 13" src="https://github.com/user-attachments/assets/50ded574-d21e-4a8f-8afd-898b38ae91a2" />

---

<img width="821" height="371" alt="Screenshot 2025-12-04 at 16 04 26" src="https://github.com/user-attachments/assets/f588ce54-d8c1-4715-af36-cd5d8cf26333" />

Deployed custom constraint template + constraint:

#### ❌ **No Privileged Containers Allowed**

Constraint Template:

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: K8sPrivilegedContainer
metadata:
  name: k8sprivilegedcontainer
spec:
  crd:
    spec:
      names:
        kind: K8sPrivilegedContainer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sprivilegedcontainer
        violation[{"msg": msg}] {
          input.review.object.spec.containers[_].securityContext.privileged
          msg := "Privileged container not allowed"
        }
```

Constraint:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPrivilegedContainer
metadata:
  name: no-privileged-containers
spec:
  enforcementAction: deny
```

---

## 🔥 **Policy Demonstration**

### 🧪 **Bad Pod Test (Blocked Successfully)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
spec:
  containers:
  - name: hacker
    image: nginx
    securityContext:
      privileged: true
```

Applied:

```bash
kubectl apply -f bad-pod.yaml
```
<img width="836" height="834" alt="Screenshot 2025-12-05 at 09 27 13" src="https://github.com/user-attachments/assets/b82ad83c-00dd-4004-a241-2af4c1480f22" />

---

<img width="827" height="557" alt="Screenshot 2025-12-05 at 09 23 16" src="https://github.com/user-attachments/assets/eba8232a-9b3c-4b05-b676-109b907cb027" />


**Result:**
❌ *"Privileged container not allowed"* — Gatekeeper blocked the pod.

<img width="839" height="93" alt="$$$$$" src="https://github.com/user-attachments/assets/746bf102-6491-4062-8ec3-e5cdcbc0afd7" />

---

<img width="823" height="176" alt="Screenshot 2025-12-05 at 09 24 01" src="https://github.com/user-attachments/assets/26989764-24b1-48ab-b693-3e75484b1f75" />

---

<img width="829" height="112" alt="Screenshot 2025-12-04 at 16 07 02" src="https://github.com/user-attachments/assets/7b76ce52-16b8-4cbc-954e-448bffb87c5c" />


---

## 📂 Project Structure

```
terraform/
│── main.tf
│── variables.tf
│── outputs.tf
│── eks.tf
│── provider.tf
│── helm-gatekeeper.tf
│── constraints/
│      ├── template.yaml
│      └── no-privileged.yaml
│── test-pods/
       ├── bad-pod.yaml
       └── good-pod.yaml
```

---

## 🛠️ **How to Deploy This Project**

### 1️⃣ Clone the repo

```bash
git clone https://github.com/<your-username>/secure-eks-vpc-gatekeeper.git
cd secure-eks-vpc-gatekeeper
```

### 2️⃣ Initialize Terraform

```bash
terraform init
terraform fmt
terraform validate
```

### 3️⃣ Deploy the infra

```bash
terraform apply
```

### 4️⃣ Update kubeconfig

```bash
aws eks update-kubeconfig --name secure-vpc-lab-eks --region us-east-1
```

### 5️⃣ Test Gatekeeper

```bash
kubectl apply -f constraints/no-privileged.yaml
kubectl apply -f test-pods/bad-pod.yaml
```

---

## 🎯 **Skills Demonstrated**

* AWS networking (VPC, subnets, NAT, IGW, routing)
* Kubernetes cluster deployment
* EKS architecture & IAM roles
* Helm chart management
* Gatekeeper / OPA policy enforcement
* Observability with VPC flow logs
* Infrastructure as Code (Terraform)
* Secure cloud-native architecture design

---

## ⭐ **Why This Project Matters**

This repository demonstrates my ability to:

* Build secure-by-default cloud infrastructure
* Enforce security controls at the Kubernetes admission layer
* Automate everything with Terraform
* Monitor and analyze network flows
* Prevent misconfigurations and privilege escalation

This is a **top 1% portfolio project** for cloud security at a professional level.

---

## 🙌 Connect With Me

If you'd like to collaborate, discuss cloud security labs, or talk about EKS/Gatekeeper security patterns - let's connect!
