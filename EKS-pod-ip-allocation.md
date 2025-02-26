# How Pods Get IP Addresses in AWS EKS

## 1. AWS VPC CNI Plugin (Primary Method)
AWS EKS uses the **Amazon VPC CNI plugin** to assign **Elastic Network Interfaces (ENIs)** and **IP addresses** from the **Amazon VPC subnet** directly to Pods.

Each **worker node** in the EKS cluster is an **EC2 instance** with one or more **ENIs**.

### 2. IP Allocation Process
1. **Node Starts & Joins the Cluster**  
   - When an EKS node (EC2 instance) joins the cluster, the **VPC CNI plugin** requests additional secondary IP addresses from the VPC subnet.

2. **Pod Scheduling**  
   - When a Pod is scheduled on a node, the **kubelet** requests an IP address from the VPC CNI plugin.

3. **IP Address Assignment**  
   - The **VPC CNI plugin** assigns a **secondary IP address** from the ENI to the Pod.
   - The Pod now has an IP address that is **directly routable within the VPC**.

### 3. How Many IPs Can a Node Handle?
- The **number of Pods per node** depends on:
  - The **instance type** (e.g., `t3.medium`, `m5.large`).
  - The **number of ENIs** it supports.
  - The **number of secondary IPs per ENI**.

For example, an **m5.large** instance supports:
- **3 ENIs** (1 primary + 2 additional).
- **10 secondary IPs per ENI**.
- Maximum **29 Pods** (since 1 IP is used by the node itself).

### 4. Alternative Networking Options
- **Calico**: Used for network policies with VPC CNI.
- **Cilium**: Uses eBPF for enhanced networking.

### 5. Viewing Pod IPs in EKS
To check the IPs assigned to Pods:
```sh
kubectl get pods -o wide
```
### 6. Example: Pod Capacity for m5.xlarge in AWS

## Overview
The number of pods an **m5.xlarge** EC2 instance can support depends on:

1. **Kubernetes vCPU-Based Pod Limit**
   - Formula:  
     ```
     max_pods = (vCPUs * 2) + 2
     ```
   - **m5.xlarge** has **4 vCPUs**, so:  
     ```
     (4 * 2) + 2 = 10 pods
     ```
   - This is the default limit but can be adjusted in **Amazon EKS**.

2. **AWS ENI & IP-Based Pod Limit**
   - **m5.xlarge** supports **3 ENIs**.
   - Each ENI provides **15 IP addresses**.
   - AWS allows pods per instance as:  
     ```
     (ENIs * IPs per ENI) - 1
     ```
   - Calculation:  
     ```
     (3 * 15) - 1 = 44 pods
     ```
   - **The primary instance IP is reserved, reducing the count by 1.**

## Final Pod Limit:
| Limit Type            | Max Pods |
|-----------------------|---------|
| Kubernetes Default    | 10      |
| AWS ENI-Based Limit   | 44      |
| **Effective Limit**   | **10 (default) or 44 (custom)** |

> **Note:** The effective limit is **the lower of the two** unless overridden in **Amazon EKS**.

## Custom Configuration
- To increase the limit beyond 10, adjust the **max pods** setting in **Amazon EKS**.
- Consider **networking constraints** when increasing pod limits.

---
