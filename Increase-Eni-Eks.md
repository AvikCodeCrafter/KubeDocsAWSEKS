# 🚀 Increasing ENI for Nodes in an EKS Cluster

In Amazon EKS (Elastic Kubernetes Service), the number of **Elastic Network Interfaces (ENIs)** and **IP addresses per ENI** determines how many pods can be deployed on a node. To increase the number of ENIs for nodes in an EKS cluster, follow these steps:

---

## **1. Use Larger EC2 Instances**
Each EC2 instance type has a limit on:
- **Number of ENIs**
- **Number of IPs per ENI**

Upgrading to a larger instance type increases both limits. You can check the maximum ENIs per instance type in the AWS documentation:  
🔗 [EC2 Instance Limits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI)

### **Instance Type Limits**
| Instance Type | Max ENIs | IPs per ENI | Max Pods (w/AWS VPC CNI) |
|--------------|---------|------------|-------------------------|
| `t3.medium`  | 3       | 6          | 17                      |
| `m5.large`   | 3       | 10         | 29                      |
| `c5.4xlarge` | 8       | 30         | 234                      |

---

## **2. Tune the VPC CNI Plugin**
The **AWS VPC CNI** plugin controls how pods get IP addresses. You can modify these settings:

### **Increase `WARM_ENI_TARGET`**  
Ensures spare ENIs are always available.
```sh
kubectl set env daemonset/aws-node -n kube-system WARM_ENI_TARGET=2
```

### **Enable Prefix Delegation (Recommended)**  
Allows assigning **16 IPs per ENI** instead of the default limit.
```sh
kubectl set env daemonset/aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true
```

**Benefit:** Increases pod density per node without needing extra ENIs.

---

## **3. Use Custom Networking (Secondary CIDR)**
By default, the AWS VPC CNI assigns pod IPs from the node’s **primary VPC CIDR block**. If you’re running out of IPs:
- **Add a secondary CIDR block to your VPC**
- **Assign pods from the secondary CIDR using custom networking**

### **Steps:**
1. **Add a secondary CIDR:**
   ```sh
   aws ec2 associate-vpc-cidr-block --vpc-id vpc-xxxx --cidr-block 100.64.0.0/16
   ```
2. **Create a new subnet in the secondary CIDR**
3. **Use `AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true`** in `aws-node`
4. **Manually assign ENIs to secondary subnets**  

🔗 [AWS Docs on Custom Networking](https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network.html)

---

## **4. Use IPv6 (Unlimited IPs)**
If you're running out of IPv4 addresses, AWS supports **IPv6 for EKS**. This removes the ENI IP limits entirely.

### **Enable IPv6 in EKS:**
- Create an **IPv6-enabled VPC** and subnets
- Set `ENABLE_IPV6=true` in the VPC CNI config

🔗 [AWS Docs on IPv6](https://docs.aws.amazon.com/eks/latest/userguide/cni-ipv6.html)

---

## **Summary of Best Approaches**
| Approach                        | Benefit                                    | Complexity |
|---------------------------------|--------------------------------|------------|
| **Use larger instance types**   | More ENIs & IPs per ENI        | Low        |
| **Enable Prefix Delegation**    | More IPs per ENI (up to 16)    | Low        |
| **Increase WARM_ENI_TARGET**    | Pre-allocate ENIs efficiently  | Medium     |
| **Use Custom Networking**       | Use secondary VPC CIDRs        | High       |
| **Enable IPv6**                 | Unlimited pod IPs              | High       |

---


