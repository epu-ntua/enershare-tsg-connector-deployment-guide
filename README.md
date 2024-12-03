# TSG/NTUA Connector Configuration Guide

Welcome to the **TSG IDS Connector Deployment Guide**. This repository provides detailed instructions for deploying and configuring the TSG IDS Connector for the **NTUA** specifications within the **Enershare** project. Follow the guide step-by-step for a seamless installation, configuration, and interaction process.

### 1. [Home](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/1.-Home)  
   **Overview and introduction** to the TSG/NTUA Connector project. This page explains the purpose of the connector, the Enershare project, and the key components involved in the deployment.


### 2. [Requirements](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/2.-Requirements)  
   **System prerequisites** for deploying the TSG IDS Connector, including hardware specifications, operating system requirements (e.g., Ubuntu), and necessary tools such as microk8s and Helm. This page also covers the minimum software versions required for deployment.


### 3. [Prerequisites](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/3.-Prerequisites)  
   **Environment setup instructions**, including installing microk8s, enabling components like ingress and cert-manager, configuring DNS, and preparing Helm for use in your Kubernetes cluster.


### 4. [Deployment](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/4.-Deployment)  
   **Step-by-step deployment instructions** for the TSG IDS Connector into your Kubernetes cluster. This includes setting up Helm, modifying the `values.yaml` file, creating identity secrets, and installing the Helm chart for the connector.


### 5. [Interacting](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/5.-Interacting)  
   **How to interact** with the deployed TSG IDS Connector and data apps. This page explains how to access the UI for managing the connector and interact programmatically through the OpenAPI for data exchange.

### 6. [Cleanup](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/6.-Cleanup)  
   **Instructions for cleaning up** and removing the connector and its resources from your Kubernetes cluster. This includes deleting Helm releases, identity secrets, cluster issuers, and other components related to the connector.

### 7. [Artifacts](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Artifacts)  
   **Managing artifacts** within the TSG IDS Connector. This section covers uploading artifacts, configuring access contracts, and managing the artifact lifecycle through the connector’s app store.

### 8. [Links](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Links)  
   A collection of **useful external links** related to the TSG IDS Connector and Enershare project, including links to Identity Providers, Metadata Brokers, and official resources for further reading.

### 9. [Persistent Storage Artifacts](https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide/wiki/Persistent%20Storage%20Artifacts)  
   **Configuring persistent storage** to retain artifacts even after container restarts. This section covers how to set up persistent volume claims (PVCs) for the TSG IDS Connector within Kubernetes, ensuring data durability.
