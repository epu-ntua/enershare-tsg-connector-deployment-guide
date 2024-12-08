# **TSG IDS Connector Deployment Guide**  
Welcome to the **TSG IDS Connector Deployment Guide**! This guide provides a comprehensive walkthrough for deploying and configuring the **TSG IDS Connector** tailored to the **NTUA specifications** as part of the **Enershare project**. Follow these steps to ensure a smooth setup and interaction with the connector.

---

## 📖 **Contents and Details**

### [1️⃣ Home 🏠](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/1.-Home)  
Learn about:  
- The purpose of the connector.  
- Its role in the Enershare project.  
- Key components involved in deployment.  

---

### [2️⃣ Requirements 📋](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/2.-Requirements)  
Check your system's compatibility:  
- **Operating System**: Ubuntu.  
- Tools like **microk8s** and **Helm**.  
- Minimum software versions for deployment.  

---

### [3️⃣ Prerequisites ⚙️](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/3.-Prerequisites)  
Set up the environment by:  
- Installing microk8s and enabling components like ingress and cert-manager.  
- Configuring DNS.  
- Preparing Helm for Kubernetes cluster management.  

---

### [4️⃣ Deployment 🚀](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/4.-Deployment)  
Deploy the TSG IDS Connector by:  
1. Setting up Helm.  
2. Modifying the `values.yaml` file to match your specifications.  
3. Creating identity secrets.  
4. Installing the Helm chart.  

---

### [5️⃣ Interacting 💬](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/5.-Interacting)  
Access and interact with your connector:  
- Manage the connector through its **UI**.  
- Exchange data programmatically using the **OpenAPI**.  

---

### [6️⃣ Cleanup 🧹](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/6.-Cleanup)  
Remove the connector and its resources by:  
- Deleting Helm releases and identity secrets.  
- Cleaning up cluster issuers and other components.  

---

### [7️⃣ Artifacts 🗂️](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Artifacts)  
Manage artifacts within the connector:  
- Upload and configure access contracts.  
- Oversee the artifact lifecycle using the app store.  

---

### [8️⃣ Links 🔗](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Links)  
Discover useful resources, including:  
- Identity Providers.  
- Metadata Brokers.  
- Official guides and further reading.  

---

### [9️⃣ Persistent Storage Artifacts 💾](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Persistent%20Storage%20Artifacts)  
Ensure data durability by:  
- Setting up Persistent Volume Claims (PVCs).  
- Retaining artifacts even after container restarts.  

---
