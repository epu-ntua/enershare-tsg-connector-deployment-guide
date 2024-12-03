# TSG/NTUA Connector Configuration Guide

This repository contains the installation instructions for the TSG IDS Connector customized to the specifications of NTUA as part of the Enershare project. 
It is the basis for deploying and configuring the components related to the connector (core container, administration UI, and data apps). The guide has been heavily inspired by TSG's [guide](https://gitlab.com/tno-tsg/projects/enershare).

## Requirements

### 1. Setup deployment environment.

Install the [microk8s](https://microk8s.io/) system. This requires:
- An Ubuntu 22.04 LTS, 20.04 LTS, 18.04 LTS or 16.04 LTS environment to run the commands (or another operating system which supports snapd - see the snapd documentation)
- At least 540MB of memory, but to accommodate workloads, it is recommended a system with at least 20G of disk space and 4G of memory.
- An internet connection

See more details regarding its configuration in the [Prerequisites](#prerequisites) section

### 2. Request dataspace certificates

You need to become a participant in a dataspace as well as create your connector credentials in the [Εnershare](https://daps.enershare.dataspac.es/#home) (or [TSG Playground](https://daps.playground.dataspac.es/#management)) dataspace. This is important to acquire the necessary certificate files and keys, as well as connector/partificant IDs (used in is secrets and `values.yaml` respectively). 
1. Create an account at the Enershare Identity Provider
2. Go to the sub-tab `Participants` within the `Management` tab and `request a Participant certificate` via the button at the bottom of the page. You can choose your own participant ID, our suggestion is to use an identifier in the form of `urn:ids:enershare:participants:ORGANISATION_NAME` (where spaces can be replaced with hypens). You can either choose to create a private key and certificate signing request via the OpenSSL CLI or use the form under `Create private key in browser`. When using the form to generate the private key, ensure you download it, since it won't be stored anywhere by default.
3. Go to the sub-tab `Connectors` withing the `Management` tab and `request a Connector certificate` via the button at the bottom of the page. The process is similar to the Participant certificate, but it requires a participant certificate. For the connector ID, our suggestion is to use an identifier in the form of `urn:ids:enershare:connectors:ORGANISATION_NAME:CONNECTOR_NAME` (where spaces can be replaced with hyphens). The connector name can be a human readable name for the connector. 

At the end of this step, a participant and connector (with appropriate IDs) should be registered and the following files should be place in the directory of your connector:  
```bash
├── cachain.crt     # certificate authority key
├── component.crt   # connector id certificate
├── component.key   # connector id key
├── participant.crt # participant/organization id certificate
└── participant.key # participant/organization id key
```    

Send a message to Maarten and Willem, or ask during one of the calls, to activate the certificates.

### 3. SwaggerHub (Optional)

(For provider connectors) you need to have uploaded your API's documentation in [SwaggerHub](https://app.swaggerhub.com/home), when deploying APIs through the connector. See more details regarding its use in the [Deployment](#deployment) section

**Note:** As of version 2.3.1 of the OpenAPI data app (image docker.nexus.dataspac.es/data-apps/openapi-data-app:2.3.1), it is no longer necessery to add your openAPI description to Swaggerhub for the connector to find your app. 
In `values.yaml` file, at both places where the `openApiBaseUrl` is allowed (on the root config of the data app and per agent) now also `openApiMapping` is supported. The structure is similar to `backendUrlMapping`, so per version the full URL of the OpenAPI document can be provided, e.g.:
```yaml
openApiMapping:
  ${api-version}: http://path_to_api_description_json
versions: 
  - ${api-version}
```

## Prerequisites

#### 1. **Install microk8s system using `snap`:** 
We use microk8s since it provides easy installation for many important compoments for the connector (kubectl, helm, ingress, cert-manager)
```bash
sudo snap install microk8s --classic
microk8s status --wait-ready # check status of microk8s cluster
```

Optionally, add the following quality-of-life commands: 
```bash
alias mkctl="microk8s kubectl" # set alias for kubectl
sudo usermod -a -G microk8s ${USER} # set sudo user for microk8s 
```

#### 2. Ensure which ports are already opened in your current working enviroment using:
```bash
sudo ufw status numbered # check port status
```

#### 3. **Enable ingress addon:** 
An ingress controller acts as a reverse proxy and load balancer. It adds a layer of abstraction to traffic routing, accepting traffic from outside the Kubernetes platform and load balancing it to Pods running inside the platform
```bash
# cert-manager requires ports 80,443 to be allowed from the firewall
sudo ufw allow 80 
sudo ufw allow 443
sudo microk8s enable ingress
```

#### 4. **Enable cert-manager addon:** 
Cert-manager is a tool for Kubernetes that makes it easy to get and manage security certificates for your websites or applications. Cert-manager talks to certificate authorities (like Let's Encrypt) automatically to get certificates for your domain.
```bash
# cert-manager requires port 9402 to be allowed from the firewall
sudo ufw allow 9402
# It might require to lower the firewall at the initial installation  
# sudo ufw disable
sudo microk8s enable cert-manager
# sudo ufw enable
```

#### 5. **Configure clusterIssuer:** 
ClusterIssuer is a Kubernetes resource that represents a specific certificate authority or a way to obtain certificates for cluster-wide issuance. It uses the ACME protocol to interact with certificate authorities (e.g Let's Encrypt) and automate the certificate issuance process.
Apply `cluster-issuer.yaml` file provided using:
```bash
microk8s kubectl apply -f cluster-issuer.yaml
```

#### 6. DNS A records 
Ensure DNS A records (or a wildcard DNS A record) is set to point to the public IP address of the VM.
```bash
# This returns VM's public IP address in ipv4 (e.g 147.102.6.27)
curl -4 ifconfig.co 
# Display the resolved IP address associated with the domain name. See if it matches output of VM's public IP address
nslookup {domain-name}
```

#### 7. Ensure dns ports (53/9153) are available:
```bash
sudo ufw allow 9153 
sudo ufw allow 53
```
#### 8. **Enable Helm addon:** 
[Helm](https://helm.sh/docs/intro/using_helm/) is a package manager software for Kubernetes applications. If not installed by default when initializing microk8s cluster, enable it manually:
 ```bash
  sudo microk8s enable helm
 ```

## Deployment

### 1. Configure the Helm Chart
Update the `values.yaml` file with the modifications to the configuration (see `/examples/values.ntua.yml` as an example).In this guide, it is assumed that you have followed the instructions in the [Requirements](#requirements) section. Please refer to the official TSG gitlab [page](https://gitlab.com/tno-tsg/helm-charts/connector/-/blob/master/README.md?ref_type=heads) for further information with regards to the configuration.The minimal configuration required to get your first deployment running, without data apps and ingresses, is as follows:

#### a. Host
Modify `host` to the domain name you configured with the ingress controller:
```yaml
host: ${domain-name}
```
#### b. Connector IDs
Modify `ids.info.idsid`, `ids.info.curator`, `ids.info.maintainer` in the `values.yml` file to the corresponding identifiers that you filled in during creation of the certificates. `ids.info.idsid` should be the Connector ID, and `ids.info.curator`, `ids.info.maintainer` should be the Participant ID. (Optionally) change `titles`and `descriptions` to the connector name, and a more descriptive description of your service in the future:
```yaml
ids:
  info:
    idsid: ${IDS_COMPONENT_ID}
    curator: ${IDS_PARTICIPANT_ID}
    maintainer: ${IDS_PARTICIPANT_ID}
    titles:
      - ${CONNECTOR TITLE@en}
    descriptions:
      - ${CONNECTOR DESCRIPTION@en}
```
#### c. Data-app agents
Modify fields in the `agents` tab: Keep in mind that `api-version` is the version number you have used for your API when you uploaded in [SwaggerHub](#3-swaggerhub) (e.g 0.5). It is important to note that in order to retrieve the API spec for the data app, the URL used in the config should be the `/apiproxy/registry/` variant instead of the `/apis/` link from Swagger hub.
    
**Important Note:** As of version 2.3.1 of the OpenAPI data app (`image docker.nexus.dataspac.es/data-apps/openapi-data-app:2.3.1`), it is no longer necessery to add your openAPI description to Swaggerhub for the connector to find your app. In `values.yaml` file, at both places where the `openApiBaseUrl` is allowed (on the root config of the data app and per agent) now also `openApiMapping` is supported. We encourage the use of `openApiMapping` for less complexity and third-party overhead. The structure is similar to `backendUrlMapping`, so per version the full URL of the OpenAPI document can be provided:
  ```yaml
  agents:
      - id: ${IDS_COMPONENT_ID}:${AgentName} # custom agent defined by user
        backEndUrlMapping:
          ${api-version}: http://${service-name}:${internal-service-port}
        title: SERVICE TITLE
        # Comment/Uncomment either openApiBaseUrl or openApiMapping snippet
        # openApiBaseUrl: https://app.swaggerhub.com/apiproxy/registry/${username}/${api-name}/
        openApiMapping:
          ${api-version}: http://path_to_api_description_json
        versions: 
        - ${api-version}
  ```
#### d. Multiple connectors (optional)
**When using multiple connectors in the same cluster**: deploy connectors at different namespaces to avoid confusion between their certificates. Each connector namespace must contain the connector helm chart as well as its respective identity-secret. Additionally, to avoid overlap between connectors in the same namespace or/and domain, you should also modify in the `values.yaml`:

- the data-app path at `containers.services.ingress.path`
    ```yaml
     services:
       - port: 8080
         name: http
         ingress:
           path: /${data-app}/(.*)
    ```

- the name of the identity secret at `coreContainer.secrets.idsIdentity.name`
    ```yaml
     secrets:
       idsIdentity:
         name: ${ids-identity-secret}
    ```

- the ingress path at `coreContainer.ingress.path` and `adminUI.ingress.path`
        ```yaml
        ingress:
            path: /${deployment-name}/ui/(.*)
            rewriteTarget: /$1
            clusterIssuer: letsencrypt
            ingressClass: public
        ```     
#### e. Security (Optional)
- Modify `ids.security.apiKey.key` and `Containers.key` fields: Change the bit after ``APIKEY-`` to a random API key used for interaction between the core container and the data app.
  ```yaml
  key: APIKEY-sgqgCPJWgQjmMWrKLAmkETDE
  ...
  apiKey: APIKEY-sgqgCPJWgQjmMWrKLAmkETDE
  ```

- Modify `ids.security.users.password` field: Create your own BCrypt encoded password for the admin user of the connector (also used in the default configuration to secure the ingress of the data app). 
  ```yaml
  users:
      - id: admin
          # -- BCrypt encoded password
          password: ${admin-password}
          roles:
              - ADMIN
  ```
- Connector's container UI is secured using ingress authentication, with credentials found in `ids.security.users` in connector's `values.yaml`. To secure the connector's data-app as well, the developer must uncomment the     `containers.services.ingress.annotations` fields in `values.yaml`. Since, the authentication is implemented using ingress, the paths' prefix must match the one in `coreContainer.ingress.path`
  ```yaml
  coreContainer.ingress.path: /${deployment-name}/(.*)
  ...
  containers.services.ingress.annotations:
    nginx.ingress.kubernetes.io/auth-url: "https://$host/${deployment-name}/external-auth/auth"
    nginx.ingress.kubernetes.io/auth-signin: "https://$host/${deployment-name}/external-auth/signin?rd=$escaped_request_uri"
  ```

### 2. Create IDS Identity secret
Cert-manager stores TLS certificates as Kubernetes secrets, making them easily accessible to your applications. When certificates are renewed, the updated certificates are automatically stored in the corresponding secrets. Create an Kubernetes secret containing the certificates acquired from identity creation.
```bash
microk8s kubectl create secret generic ids-identity-secret --from-file=ids.crt=./component.crt \
                                                           --from-file=ids.key=./component.key \
                                                           --from-file=ca.crt=./cachain.crt    \
                                                           -n ${namespace} 
```
please update to appropriate names the `namespace` (e.g default)
   
### 3. Add the Helm repository
Add the Helm repository of the TSG components:
```bash
helm repo add tsg https://nexus.dataspac.es/repository/tsg-helm
helm repo update
```

### 4. Helm chart installation
To install the Helm chart, execute:
```bash
microk8s helm upgrade --install                               \
        -n ${namespace}                                       \
        --repo https://nexus.dataspac.es/repository/tsg-helm  \
        --version 3.2.8                                       \
        -f values.yaml                                        \
        ${deployment-name}                                    \
        tsg-connector
```
please update to appropriate names the `namespace` (e.g default) and `deployment-name` (e.g my-connector) fields

### 5. Wait
Wait till you ensure connector pods are all in a ready (1/1) state (it might take at least a minute). You can watch the state of the pods using this command:
```bash
watch microk8s kubectl get all --all-namespaces
```  

## Interacting

The connector address that other connectors will use to communicate with your connector will be `https://${domain-name}/router`. 

Also, after successful deployment, your connector should be available in the [Metadata Broker](https://broker.enershare.dataspac.es/#connectors).

### GUI
After deployment, the user interfaces for the : 
- data space connector (`https://${domain-name}/${deployment-name}/ui/`)
- connector data-app (`https://${domain-name}/${data-app}/`)

will be available, with the login matching the admin user with the provided BCrypt password. 

### Programmatically

To utilize the functionality of the Open API data app programmatically, you can make client side calls to query the app. This is the same method used by the app's user interface.
For a more concrete template example, please look at `/examples/client-app.py`, where we use [python requests](https://requests.readthedocs.io/en/latest/user/quickstart/) to perform API calls to the connector. Please note that the script might not work at the time of your execution, as the credentials will be updated.

#### a. URL
The structure of these calls is as follows: `https://<baseurl>/<data-app-path>/openapi/<version>/<endpoint>`. 
The baseurl represents the URL where your connector is deployed, while the `data-app-path` refers to the path used for the data app. 
For the `version`, you should select the version of your own backend service, and for the endpoint, choose an endpoint specific to your service.

#### b. Headers
To ensure that the OpenAPI data app knows where to route the request, you can include headers with the request. The headers used are the:
- `Authorization`:  the Bearer Authentication HTTP header, so the field is filled as `Bearer`, plus space, plus the API key defined in `values.yaml` file at `containers.apiKey` (incluing `APIKEY-` prefix). This header field is required **only** if data-app UI is configured with ingress authentication.
- `Forward-ID`: the Agent ID of the service registered at the party you wish to interact with (reciever)
- `Forward-Sender`: your own Agent ID for identification purposes (`${IDS_COMPONENT_ID}`)

#### c. Params
In case of Query parameters are not URL encoded and therefore, if you need to perform queries, require to use the `params` option.
If you are using external authentication, it is advisable not to make calls to the OpenAPI data app via the ingress. 
In such cases, you can deploy your service in the same Kubernetes cluster as the data app and use the internal Kubernetes service URL to access the data app.

## Usage

In the OpenAPI data app UI:
1. go to `Tester` and click on `Query`
2. expand the agent with id `urn:ids:enershare:connectors:MeterDataService:ServiceAgent` and click on `Use`. The fields appropriate to said agent should be filled
3. Select a sender agent from the list and provide as `path` "/powermeters"
4. This should result in a JSON array of observer Ids.

## Clean-up

To delete the connector and remove all related resources:
```bash
microk8s kubectl delete clusterissuer letsencrypt -n ${namespace}
microk8s kubectl delete secret/${ids-identity-secret} -n ${namespace}
microk8s helm uninstall ${deployment-name} -n ${namespace}
```

## Links

1. Reference docs: https://github.com/european-dynamics-rnd/enershare-ds-components​
2. Identity Provider: https://daps.enershare.dataspac.es/#connectors​
3. Metadata Broker: https://broker.enershare.dataspac.es/#connectors​
4. Appstore: https://store.haslab-dataspace.pt/gui/#/auth/login ​
5. Marketplace: https://enershare.zapto.org:4173/​
6. Vocabulary Hub: https://energy.vocabulary-hub.eu/specifications ​
7. Clearing House: https://enershare.eurodyn.com/dashboard ​
8. TSG Connector Installation: https://github.com/epu-ntua/enershare-tsg-connector-deployment-guide ​

## Steps to Upload Artifacts

1. **Access the Connector UI**
   - Navigate to the connector UI at `/ui`.
   - Log in using your credentials.

2. **Navigate to the Resources Tab**
   - After logging in, go to the "Resources" tab.

3. **Add Artifact Details**
   - Enter the **Title** and **Description** for your artifact.
   - Select the artifact from your local system by clicking the "Upload" button.

4. **Define Access Contract**
   - Specify the contract that will govern access to the artifact, or select the "Generic Read Access" option for default access settings.

5. **Upload the Artifact**
   - Once all details are filled in, click the "Upload" button to upload the artifact.
   - The artifact is now hosted by the container and will be automatically listed in the Appstore.

> **Important:** If the connector pods fail, the uploaded artifact will be lost.

# TSG/NTUA Connector Configuration Guide

This repository provides the setup instructions for the TSG IDS Connector for the NTUA specifications in the Enershare project. This guide includes all the necessary steps for deploying and configuring the TSG IDS Connector, including installation, interaction, and artifact management.

## Table of Contents

- [Home](./wiki/Home)
- [Requirements](./wiki/Requirements)
- [Prerequisites](./wiki/Prerequisites)
- [Deployment](./wiki/Deployment)
- [Interacting](./wiki/Interacting)
- [Cleanup](./wiki/Cleanup)
- [Artifacts](./wiki/Artifacts)
- [Links](./wiki/Links)
- [Persistent Storage Artifacts](./wiki/Persistent%20Storage%20Artifacts)

## Overview

This guide covers the steps required to deploy and configure the TSG IDS Connector in the NTUA setup. Each section in the repository provides in-depth instructions for different aspects of the setup and operation.

### [Home](./wiki/Home)
The "Home" page offers an overview and general introduction to the TSG/NTUA Connector project. It briefly explains the purpose of the connector, the Enershare project, and the key components involved in the deployment.

### [Requirements](./wiki/Requirements)
The "Requirements" page lists all the system prerequisites needed for deploying the TSG IDS Connector. This includes the necessary hardware, operating systems (e.g., Ubuntu), and tools (e.g., microk8s, Helm). It also highlights the minimum system specifications and software versions required to ensure a smooth deployment.

### [Prerequisites](./wiki/Prerequisites)
On the "Prerequisites" page, you'll find instructions to set up the environment for deploying the connector. This section covers the installation of microk8s, enabling essential components such as ingress and cert-manager, and ensuring correct DNS configurations. It also provides guidance on preparing Helm for use with the connector.

### [Deployment](./wiki/Deployment)
The "Deployment" page contains step-by-step instructions for deploying the TSG IDS Connector. It guides you through the installation process, including:
- Setting up Helm.
- Configuring the `values.yaml` file for the connector.
- Creating secrets for identity certificates.
- Installing the Helm chart that will deploy the connector into your Kubernetes cluster.

### [Interacting](./wiki/Interacting)
The "Interacting" page explains how to interact with the deployed TSG IDS Connector. You’ll learn how to access the connector’s UI for managing the connector and data apps. It also provides details on programmatically interacting with the connector via the OpenAPI data app using Python or other methods.

### [Cleanup](./wiki/Cleanup)
The "Cleanup" page provides instructions on how to clean up and remove the connector and associated resources when no longer needed. This includes deleting the Helm release, identity secrets, cluster issuers, and any other Kubernetes resources tied to the deployment.

### [Artifacts](./wiki/Artifacts)
The "Artifacts" page provides instructions for managing artifacts within the TSG IDS Connector. You’ll learn how to upload artifacts, configure access contracts, and manage them through the connector's appstore. This section also explains how to interact with and track the lifecycle of artifacts.

### [Links](./wiki/Links)
The "Links" page contains a collection of useful external references related to the TSG IDS Connector and the Enershare project. It includes links to resources such as the Identity Provider, Metadata Broker, and the official TSG connector installation guide, among others.

### [Persistent Storage Artifacts](./wiki/Persistent%20Storage%20Artifacts)
The "Persistent Storage Artifacts" page explains how to configure persistent storage to retain artifacts even if the containers are restarted. It covers how to set up persistent volume claims (PVC) for the connector and manage storage in Kubernetes to ensure that artifacts are not lost during deployments or restarts.

For more detailed instructions on any of these topics, refer to the corresponding page in the wiki.
