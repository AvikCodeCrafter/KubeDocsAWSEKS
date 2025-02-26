# **How CNI Works in AWS EKS Cluster**

## **1. What is CNI in AWS EKS?**
Container Network Interface (**CNI**) is a networking framework that allows container orchestrators like Kubernetes to configure network interfaces dynamically. In AWS Elastic Kubernetes Service (**EKS**), the **Amazon VPC CNI plugin** is used as the default networking solution. This plugin allows Kubernetes pods to get IP addresses directly from the Amazon **VPC (Virtual Private Cloud)**, enabling seamless communication between pods and other AWS resources.

---

## **2. AWS VPC CNI Plugin Overview**
The **AWS VPC CNI Plugin** is a Kubernetes CNI plugin that allows pods to use native Amazon VPC networking. It:
- Assigns an **Elastic Network Interface (ENI)** from the worker node's VPC subnet.
- Allocates **IP addresses** to pods from the VPC subnet.
- Supports **high-performance networking** features like AWS Nitro, SR-IOV, and enhanced networking.

---

## **3. How AWS VPC CNI Works**
When a new pod is created in an EKS cluster, the **CNI plugin** follows these steps:

### **Step 1: ENI and IP Address Allocation**
1. The VPC CNI plugin runs as a **DaemonSet** (`aws-node`) on each worker node.
2. The plugin checks the **available ENIs** and **IP addresses**.
3. If needed, the plugin **attaches a new ENI** to the node using AWS APIs.
4. It then assigns **secondary private IP addresses** to the ENI from the VPC CIDR range.
5. These IPs are stored in the **IP address pool**.

### **Step 2: Assigning IP Addresses to Pods**
1. When a new pod is scheduled on a worker node:
   - The kubelet requests an IP address from the CNI plugin.
   - The CNI plugin selects an available IP from the **IP pool**.
   - It configures the network interface in the pod’s network namespace.
   - It updates Kubernetes with the pod’s assigned IP.

### **Step 3: Routing and Pod Communication**
- Since pods use **VPC-native IP addresses**, they communicate directly using AWS networking.
- There is **no need for NAT or overlay networks** like Flannel or Calico.
- The ENIs are attached to EC2 instances, and traffic flows through the **VPC routing tables**.

---

## **4. AWS VPC CNI Configuration**
The VPC CNI plugin is controlled by environment variables in the **`aws-node` DaemonSet**. Some key settings include:

| Environment Variable | Description |
|----------------------|-------------|
| `WARM_ENI_TARGET` | Number of extra ENIs to keep available |
| `WARM_IP_TARGET` | Number of extra IPs to keep pre-allocated |
| `MINIMUM_IP_TARGET` | Minimum number of IPs to allocate |
| `MAX_ENI` | Maximum number of ENIs a node can allocate |

To check the current configuration:
```sh
kubectl get daemonset aws-node -n kube-system -o yaml
```

To modify a setting:
```sh
kubectl edit daemonset aws-node -n kube-system
```

---

## **5. IP Address Management in AWS VPC CNI**
The number of IPs assigned per node depends on the **instance type**. Each instance has:
- A **primary ENI** (used for node-level networking).
- Multiple **secondary ENIs** (used for pods).
- Each ENI can have a fixed number of **secondary IP addresses**.

For example, an **m5.large** instance has:
- **3 ENIs**
- **10 IPs per ENI**
- **(3 ENIs x 10 IPs) - 1 primary IP = 29 pods per node**

To check IP allocation:
```sh
kubectl exec -n kube-system aws-node-xxxxx -- ipamd --show-ipam
```

---

## **6. Scaling and Performance Optimization**
To optimize pod networking in EKS:
- **Increase `WARM_IP_TARGET`**: Ensures pre-allocated IPs are available for fast pod startup.
- **Use `PREFIX_DELEGATION`**: Enables IPv4 prefix assignment for better IP utilization.
- **Monitor IP usage**: Use `ipamd` metrics with **CloudWatch** for monitoring.

---

## **7. Limitations of AWS VPC CNI**
- **IP Exhaustion**: Limited by the number of available IPs in the VPC.
- **ENI Limits**: The number of pods per node is restricted by EC2 ENI limits.
- **Cross-Region/VPC Communication**: Requires VPC Peering or AWS Transit Gateway.

---

## **8. Alternative CNI Solutions**
While AWS VPC CNI is the default, you can use alternative CNIs:
- **Calico**: Provides Network Policies and can use BGP for routing.
- **Cilium**: Uses eBPF for high-performance networking.
- **Weave**: Supports overlay networking.

---

## **Conclusion**
The **AWS VPC CNI Plugin** integrates Kubernetes networking with AWS VPC, ensuring high performance and security. Understanding its configuration and limitations helps in optimizing network performance in an EKS cluster.

Would you like help with configuring or troubleshooting AWS VPC CNI? 🚀
