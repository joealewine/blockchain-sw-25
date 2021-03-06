---

copyright:
  years: 2018, 2020
lastupdated: "2020-06-17"

keywords: OpenShift, IBM Blockchain Platform console, deploy, resource requirements, storage, parameters

subcollection: blockchain-sw-25

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:note: .note}
{:important: .important}
{:tip: .tip}
{:pre: .pre}

# Deploying {{site.data.keyword.blockchainfull_notm}} Platform 2.5
{: #deploy-ocp}

<div style="background-color: #f4f4f4; padding-left: 20px; border-bottom: 2px solid #0f62fe; padding-top: 12px; padding-bottom: 4px; margin-bottom: 16px;">
  <p style="line-height: 10px;">
    <strong>Running a different version of IBM Blockchain Platform?</strong> Switch to version
    <a href="https://cloud.ibm.com/docs/blockchain-sw?topic=blockchain-sw-deploy-ocp">2.1.2</a>,
    <a href="https://cloud.ibm.com/docs/blockchain-sw-213?topic=blockchain-sw-213-deploy-ocp">2.1.3</a>
    </p>
</div>


You can use the following instructions to deploy the {{site.data.keyword.blockchainfull}} Platform 2.5 onto a Kubernetes cluster that is running on OpenShift Container Platform 3.11, 4.2, or 4.3. The {{site.data.keyword.blockchainfull_notm}} Platform uses a [Kubernetes Operator](https://www.openshift.com/learn/topics/operators){: external} to install the {{site.data.keyword.blockchainfull_notm}} Platform console on your cluster and manage the deployment and your blockchain nodes. After the {{site.data.keyword.blockchainfull_notm}} Platform console is running on your cluster, you can use the console to create blockchain nodes and operate a multicloud blockchain network.
{:shortdesc}




If you prefer to automate the installation of the service, check out the [Ansible Playbook](/docs/blockchain-sw-25?topic=blockchain-sw-25-ansible-install-ibp) that can be used to complete all of these steps for you.
{: tip}

## Resources required
{: #deploy-ocp-resources-required}

Ensure that your OpenShift cluster has sufficient resources for the {{site.data.keyword.blockchainfull_notm}} console and for the blockchain nodes that you create. The amount of resources that are required can vary depending on your infrastructure, network design, and performance requirements. To help you deploy a cluster of the appropriate size, the default CPU, memory, and storage requirements for each component type are provided in this table. Your actual resource allocations are visible in your blockchain console when you deploy a node and can be adjusted at deployment time or after deployment according to your business needs.

The resources for the CA, peer, and ordering nodes need to be multiplied by the number of these nodes that you require. The operator and console resources are per blockhain deployment. For example, if you deployed development, staging, and test networks in a single cluster, you need to have enough resources for three instances of the operator and console, one for each blockchain deployment. On the other hand, the webhook resources are per OpenShift cluster, only one instance is required, regardless of the number of blockchain networks in the cluster.

| **Component** (all containers) | CPU**  | Memory (GB) | Storage (GB) |
|--------------------------------|---------------|-----------------------|------------------------|
| **Peer (Hyperledger Fabric v1.4)**                       | 1.1           | 2.8                   | 200 (includes 100GB for peer and 100GB for state database)|
| **Peer (Hyperledger Fabric v2.x)**                       | 0.7           | 2.0                   | 200 (includes 100GB for peer and 100GB for state database)|
| **CA**                         | 0.1           | 0.2                   | 20                     |
| **Ordering node**              | 0.35          | 0.7                   | 100                    |
| **Operator**                   | 0.1           | 0.2                   | 0                      |
| **Console**                    | 1.2           | 2.4                   | 10                     |
| **Webhook**                    | 0.1           | 0.2                   | 0                      |

{: caption="Table 1. Default resource allocations" caption-side="bottom"}
** These values can vary slightly. Actual VPC allocations are visible in the blockchain console when a node is deployed.  

If you are deploying the OpenShift platform on {{site.data.keyword.cloud_notm}}, you must create a 4x16 cluster with multiple nodes. This cluster provides sufficient resources to get started by deploying a basic network. Note that one core in OpenShift is equivalent to one CPU (or vCPU) in {{site.data.keyword.blockchainfull_notm}} Platform.
{: important}

## Browsers
{: #deploy-ocp-browsers}
The {{site.data.keyword.blockchainfull_notm}} Platform console has been successfully tested on the following browsers:

- Chrome Version 83.0.4103.97 (Official Build) (64-bit)
- Safari Version 13.0.3 (15608.3.10.1.4)


## Storage
{: #deploy-ocp-storage}

{{site.data.keyword.blockchainfull_notm}} Platform requires persistent storage for each CA, peer, and ordering node that you deploy, in addition to the storage required by the {{site.data.keyword.blockchainfull_notm}} console. The {{site.data.keyword.blockchainfull_notm}} Platform console uses [dynamic provisioning](https://docs.openshift.com/container-platform/4.2/storage/dynamic-provisioning.html){: external} to allocate storage for each blockchain node that you deploy by using a pre-defined storage class. You have the opportunity to choose your persistent storage from the available storage options for the OpenShift Container Platform.

Before you deploy the {{site.data.keyword.blockchainfull_notm}} Platform console, you must create a storage class with enough backing storage for the {{site.data.keyword.blockchainfull_notm}} console and the nodes that you create. You can set this storage class to the default storage class of your Kubernetes cluster or create a new class that is used by the {{site.data.keyword.blockchainfull_notm}} Platform console. If you are using a multizone cluster in OpenShift Container Platform, then you must configure the default storage class for each zone. After you create the storage class, run the command `kubectl patch storageclass` to set the storage class of the multizone region to be the default storage class.

If you prefer not to choose a persistent storage option, the default storage class of your OpenShift project is used. `Host-local` volume storage is not supported.
{: note}

## Get your entitlement key
{: #deploy-ocp-entitlement-key}

When you purchase the {{site.data.keyword.blockchainfull_notm}} Platform from PPA, you receive an entitlement key for the software is associated with your MyIBM account. You need to access and save this key to deploy the platform.

1. Log in to [MyIBM Container Software Library](https://myibm.ibm.com/products-services/containerlibrary){: external} with the IBMid and password that are associated with the entitled software.

2. In the Entitlement keys section, select **Copy key** to copy the entitlement key to the clipboard. Save this value for use later during deployment.

## Before you begin
{: #deploy-ocp-prerequisites}

1. The {{site.data.keyword.blockchainfull_notm}} Platform can be installed only on the [OpenShift Container Platform 3.11, 4.2, or 4.3](https://docs.openshift.com/container-platform/3.11/welcome/index.html){: external}.

2. You need to install and connect to your cluster by using the [OpenShift Container Platform CLI](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html){: external} to deploy the platform. If you are using an OpenShift cluster that was deployed with the {{site.data.keyword.IBM_notm}} Kubernetes Service, use these instructions to [Install the OpenShift Origin CLI](/docs/openshift?topic=openshift-openshift-cli#cli_oc).

**Looking for a way to script the deployment of the service?** Check out the [Ansible playbooks](/docs/blockchain-sw-25?topic=blockchain-sw-25-ansible), a powerful tool for scripting the deployment of components in your blockchain network. If you prefer a manual installation, proceed to the next section.

## Log in to your OpenShift cluster
{: #deploy-ocp-login}

Before you can complete the next steps, you need to log in to your cluster by using the OpenShift CLI. You can log in to your cluster by using the OpenShift web console.

1. Open the OpenShift web console. If you are using the {{site.data.keyword.IBM_notm}} Kubernetes Service, you can go to your cluster by using the [{{site.data.keyword.cloud_notm}} dashboard](https://cloud.ibm.com/kubernetes/clusters){: external}. In the upper right corner of the cluster overview page, click **OpenShift web console**.

2. From the web console, click the dropdown menu in the upper right corner and then click **Copy Login Command**. Paste the copied command in your terminal window.

The command looks similar to the following example:
```
oc login https://c100-e.us-south.containers.cloud.ibm.com:31394 --token=<TOKEN>
```
If the command is successful, you can see the list of the projects in your cluster in your terminal by running the following command:
```
oc get pods
```
{:codeblock}

If successful, you can see the pods that are running in your default namespace:
```
docker-registry-7d8875c7c5-5fv5j    1/1       Running   0          7d
docker-registry-7d8875c7c5-x8dfq    1/1       Running   0          7d
registry-console-6c74fc45f9-nl5nw   1/1       Running   0          7d
router-6cc88df47c-hqjmk             1/1       Running   0          7d
router-6cc88df47c-mwzbq             1/1       Running   0          7d
```

When you connect to your cluster by using the OpenShift CLI, you also connect by using the `kubectl` CLI. You can find the same pods by running the equivalent `kubectl` command:
```
kubectl get pods
```
{:codeblock}

## Create the `ibpinfra` project for the webhook
{: #deploy-ocp-ibpinfra}

Because the platform has updated the internal apiversion from `v1alpha1` in previous versions to `v1alpha2` in 2.5, a Kubernetes conversion webhook is required to update the CA, peer, operator, and console to the new API version. This webhook will continue to be used in the future, so new deployments of the platform are required to deploy it as well.  The webhook is deployed to its own project, referred to as  `ibpinfra` throughout these instructions.

After you log in to your cluster, you can create the new `ibpinfra` project for the Kubernetes conversion webhook using the kubectl CLI. The new project needs to be created by a cluster administrator.  

Run the following command to create the project:   
```
oc new-project ibpinfra
```
{:codeblock}

When you create a new project, a new namespace is created with the same name as your project. You can verify that the existence of the new namespace by using the `oc get namespace` command:
```
$ oc get namespace
NAME                                STATUS    AGE
ibpinfra                            Active    2m
```

## Create a secret for your entitlement key
{: #deploy-ocp-secret-ibpinfra}

After you purchase the {{site.data.keyword.blockchainfull_notm}} Platform, you can access the [My IBM dashboard](https://myibm.ibm.com/dashboard/){: external} to obtain your entitlement key for the offering. You need to store the entitlement key on your cluster by creating a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/){: external}. Kubernetes secrets are used to securely store the key on your cluster and pass it to the operator and the console deployments.

Run the following command to create the secret and add it to your `ibpinfra` namespace or project:
```
kubectl create secret docker-registry docker-key-secret --docker-server=cp.icr.io --docker-username=cp --docker-password=<KEY> --docker-email=<EMAIL> -n ibpinfra
```
{:codeblock}
- Replace `<KEY>` with your entitlement key.
- Replace `<EMAIL>` with your email address.

The name of the secret that you are creating is `docker-key-secret`. It is required by the webhook that you will deploy later.
{: note}


## Deploy the webhook and custom resource definitions to your OpenShift cluster
{: #deploy-k8s-webhook-crd}

Because the platform has updated the internal apiversion from `v1alpha1` in previous versions to `v1alpha2` in 2.5, a Kubernetes conversion webhook is required to update the CA, peer, operator, and console to the new API version. This webhook will continue to be used in the future, so new deployments of the platform are required to deploy it as well.  

Before you can upgrade an existing network to 2.5, or deploy a new instance of the platform to your Kubernetes cluster, you need to create the conversion webhook by completing the steps in this section. The webhook is deployed to its own namespace or project, referred to `ibpinfra` throughout these instructions.

The first two steps are for deployment of the webhook. The last five steps are for the custom resource definitions for the CA, peer, orderer, and console components that the {{site.data.keyword.blockchainfull_notm}} requires. You only have to deploy the webhook and custom resource definitions **once per cluster**. If you have already deployed this webhook and custom resource definitions to your cluster, you can skip these seven steps below.
{: important}

### 1. Configure role-based access control (RBAC) for the webhook
{: #webhook-rbac}

First, copy the following text to a file on your local system and save the file as `rbac.yaml`. This step allows the webhook to read and create a TLS secret in its own project.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webhook
  namespace: ibpinfra
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: webhook
rules:
- apiGroups:
  - "*"
  resources:
  - secrets
  verbs:
  - "*"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: ibpinfra
subjects:
- kind: ServiceAccount
  name: webhook
  namespace: ibpinfra
roleRef:
  kind: Role
  name: webhook
  apiGroup: rbac.authorization.k8s.io
```
{: codeblock}

Run the following command to add the file to your cluster definition:
```
kubectl apply -f rbac.yaml -n ibpinfra
```
{:codeblock}

When the command completes successfully, you should see something similar to:
```
serviceaccount/webhook created
role.rbac.authorization.k8s.io/webhook created
rolebinding.rbac.authorization.k8s.io/ibpinfra created
```

### 2. Deploy the webhook
{: #webhook-deploy}

In order to deploy the webhook, you need to create two `.yaml` files and apply them to your Kubernetes cluster.

#### deployment.yaml
{: #webhook-deployment-yaml}

Copy the following text to a file on your local system and save the file as `deployment.yaml`. If you are deploying on OpenShift Container Platform v4.2 or v4.3 on LinuxONE, you need to replace `amd64` with `s390x`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "ibp-webhook"
  labels:
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibp-webhook"
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/instance: "ibp-webhook"
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        helm.sh/chart: "ibm-ibp"
        app.kubernetes.io/name: "ibp"
        app.kubernetes.io/instance: "ibp-webhook"
      annotations:
        productName: "IBM Blockchain Platform"
        productID: "54283fa24f1a4e8589964e6e92626ec4"
        productVersion: "2.5.0"
    spec:
      serviceAccountName: webhook
      imagePullSecrets:
        - name: docker-key-secret
      hostIPC: false
      hostNetwork: false
      hostPID: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
      containers:
        - name: "ibp-webhook"
          image: "cp.icr.io/cp/ibp-crdwebhook:2.5.0-20200618-amd64"
          imagePullPolicy: Always
          securityContext:
            privileged: false
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
            capabilities:
              drop:
              - ALL
              add:
              - NET_BIND_SERVICE
          env:
            - name: "LICENSE"
              value: "accept"
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: server
              containerPort: 3000
          livenessProbe:
            httpGet:
              path: /healthz
              port: server
              scheme: HTTPS
            initialDelaySeconds: 30
            timeoutSeconds: 5
            failureThreshold: 6
          readinessProbe:
            httpGet:
              path: /healthz
              port: server
              scheme: HTTPS
            initialDelaySeconds: 26
            timeoutSeconds: 5
            periodSeconds: 5
          resources:
            requests:
              cpu: 0.1
              memory: "100Mi"
```
{: codeblock}


Run the following command to add the file to your cluster definition:
```
kubectl apply -n ibpinfra -f deployment.yaml
```
{: codeblock}

When the command completes successfully, you should see something similar to:
```
deployment.apps/ibp-webhook created
```

#### service.yaml
{: #webhook-service-yaml}

Second, copy the following text to a file on your local system and save the file as `service.yaml`.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: "ibp-webhook"
  labels:
    type: "webhook"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibp-webhook"
    helm.sh/chart: "ibm-ibp"
spec:
  type: ClusterIP
  ports:
    - name: server
      port: 443
      targetPort: server
      protocol: TCP
  selector:
    app.kubernetes.io/instance: "ibp-webhook"
```
{: codeblock}

Run the following command to add the file to your cluster definition:
```
kubectl apply -n ibpinfra -f service.yaml
```
{: codeblock}

When the command completes successfully, you should see something similar to:
```
service/ibp-webhook created
```

### 3. Extract the certificate

Next, we need to extract the TLS certificate that was generated by the webhook deployment so that it can be used in the custom resource definitions in the next steps. Run the following command to extract the secret to a base64 encoded string:

```
kubectl get secret webhook-tls-cert -n ibpinfra -o json | jq -r .data.\"cert.pem\"
```
{: codeblock}

The output of this command is a base64 encoded string and looks similar to:
```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJoRENDQVNtZ0F3SUJBZ0lRZDNadkhZalN0KytKdTJXbFMvVDFzakFLQmdncWhrak9QUVFEQWpBU01SQXcKRGdZRFZRUUtFd2RKUWswZ1NVSlFNQjRYRFRJd01EWXdPVEUxTkRrME5sFORGsxTVZvdwpFakVRTUE0R0ExVUVDaE1IU1VKTklFbENVREJaTUJGcVRyV0Z4WFBhTU5mSUkrYUJ2RG9DQVFlTW3SUZvREFUQmdOVkhTVUVEREFLQmdncgpCZ0VGQlFjREFUQU1CZ05WSFJNQkFmOEVBakFBTUNvR0ExVWRFUVFqTUNHQ0gyTnlaQzEzWldKb2IyOXJMWE5sCmNuWnBZMlV1ZDJWaWFHOXZheTV6ZG1Nd0NnWUlLb1pJemowRUF3SURTUUF3UmdJaEFNb29kLy9zNGxYaTB2Y28KVjBOMTUrL0h6TkI1cTErSTJDdU9lb1c1RnR4MUFpRUEzOEFlVktPZnZSa0paN0R2THpCRFh6VmhJN2lBQVV3ZAo3ZStrOTA3TGFlTT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
```

Save the base64 encoded string that is returned by this command to be used in the next steps when you create the custom resource definitions.
{: important}

### 4. Create the CA custom resource definition
{: #deploy-crd-ca}

Copy the following text to a file on your local system and save the file as `ibpca-crd.yaml`.
```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  labels:
    app.kubernetes.io/instance: ibpca
    app.kubernetes.io/managed-by: ibp-operator
    app.kubernetes.io/name: ibp
    helm.sh/chart: ibm-ibp
    release: operator
  name: ibpcas.ibp.com
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhookClientConfig:
      service:
        namespace: ibpinfra
        name: ibp-webhook
        path: /crdconvert
      caBundle: "<CABUNDLE>"
  validation:
    openAPIV3Schema:
      x-kubernetes-preserve-unknown-fields: true    
  group: ibp.com
  names:
    kind: IBPCA
    listKind: IBPCAList
    plural: ibpcas
    singular: ibpca
  scope: Namespaced
  subresources:
    status: {}
  version: v1alpha2
  versions:
  - name: v1alpha2
    served: true
    storage: true
  - name: v210
    served: false
    storage: false
  - name: v212
    served: false
    storage: false
  - name: v1alpha1
    served: true
    storage: false
```
{:codeblock}


Replace the value of `<CABUNDLE>` with the base64 encoded string that you extracted in step three after the webhook deployment.  

Then, use the `kubectl` CLI to add the custom resource definition to your project.

```
kubectl apply -f ibpca-crd.yaml
```
{:codeblock}

You should see the following output when it is successful:
```
customresourcedefinition.apiextensions.k8s.io/ibpcas.ibp.com created
```

### 5. Create the peer custom resource definition
{: #deploy-crd-peer}

Copy the following text to a file on your local system and save the file as `ibppeer-crd.yaml`.
```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ibppeers.ibp.com
  labels:
    release: "operator"
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibppeer"
    app.kubernetes.io/managed-by: "ibp-operator"
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhookClientConfig:
      service:
        namespace: ibpinfra
        name: ibp-webhook
        path: /crdconvert
      caBundle: "<CABUNDLE>"
  validation:
    openAPIV3Schema:
      x-kubernetes-preserve-unknown-fields: true    
  group: ibp.com
  names:
    kind: IBPPeer
    listKind: IBPPeerList
    plural: ibppeers
    singular: ibppeer
  scope: Namespaced
  subresources:
    status: {}
  version: v1alpha2
  versions:
  - name: v1alpha2
    served: true
    storage: true
  - name: v1alpha1
    served: true
    storage: false
```
{:codeblock}


Replace the value of `<CABUNDLE>` with the base64 encoded string that you extracted in step three after the webhook deployment.  

Then, use the `kubectl` CLI to add the custom resource definition to your project.

```
kubectl apply -f ibppeer-crd.yaml
```
{:codeblock}

You should see the following output when it is successful:
```
customresourcedefinition.apiextensions.k8s.io/ibppeers.ibp.com created
```

### 6. Create the orderer custom resource definition
{: #deploy-crd-orderer}

Copy the following text to a file on your local system and save the file as `ibporderer-crd.yaml`.

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ibporderers.ibp.com
  labels:
    release: "operator"
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibporderer"
    app.kubernetes.io/managed-by: "ibp-operator"
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhookClientConfig:
      service:
        namespace: ibpinfra
        name: ibp-webhook
        path: /crdconvert
      caBundle: "<CABUNDLE>"
  validation:
    openAPIV3Schema:
      x-kubernetes-preserve-unknown-fields: true    
  group: ibp.com
  names:
    kind: IBPOrderer
    listKind: IBPOrdererList
    plural: ibporderers
    singular: ibporderer
  scope: Namespaced
  subresources:
    status: {}
  version: v1alpha2
  versions:
  - name: v1alpha2
    served: true
    storage: true
  - name: v1alpha1
    served: true
    storage: false
```
{:codeblock}


Replace the value of `<CABUNDLE>` with the base64 encoded string that you extracted in step three after the webhook deployment.

Then, use the `kubectl` CLI to add the custom resource definition to your project.

```
kubectl apply -f ibporderer-crd.yaml
```
{:codeblock}

You should see the following output when it is successful:
```
customresourcedefinition.apiextensions.k8s.io/ibporderers.ibp.com created
```

### 7. Create the console custom resource definition
{: #deploy-crd-console}

Copy the following text to a file on your local system and save the file as `ibpconsole-crd.yaml`.
```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ibpconsoles.ibp.com
  labels:
    release: "operator"
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibpconsole"
    app.kubernetes.io/managed-by: "ibp-operator"
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhookClientConfig:
      service:
        namespace: ibpinfra
        name: ibp-webhook
        path: /crdconvert
      caBundle: "<CABUNDLE>"
  validation:
    openAPIV3Schema:
      x-kubernetes-preserve-unknown-fields: true
  group: ibp.com
  names:
    kind: IBPConsole
    listKind: IBPConsoleList
    plural: ibpconsoles
    singular: ibpconsole
  scope: Namespaced
  subresources:
    status: {}
  version: v1alpha2
  versions:
  - name: v1alpha2
    served: true
    storage: true
  - name: v1alpha1
    served: true
    storage: false
```
{:codeblock}


Replace the value of `<CABUNDLE>` with the base64 encoded string that you extracted in step three after the webhook deployment.  
Then, use the `kubectl` CLI to add the custom resource definition to your project.

```
kubectl apply -f ibpconsole-crd.yaml
```
{:codeblock}

You should see the following output when it is successful:
```
customresourcedefinition.apiextensions.k8s.io/ibpconsoles.ibp.com created
```


## Create a new project for your {{site.data.keyword.blockchainfull_notm}} Platform deployment
{: #deploy-ocp-project}

Next, you need to create a second project for your deployment of {{site.data.keyword.blockchainfull_notm}} Platform. You can create a new project by using the OpenShift web console or OpenShift CLI. The new project needs to be created by a cluster administrator.

If you are using the CLI, create a new project by the following command:
```
oc new-project <PROJECT_NAME>
```
{:codeblock}

Replace `<PROJECT_NAME>` with the name that you want to use for your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

It is required that you create a new OpenShift project for each blockchain network that you deploy with the {{site.data.keyword.blockchainfull_notm}} Platform. For example, if you plan to create different networks for development, staging, and production, then you need to create a unique project for each environment. Each project creates a new Kubernetes namespace. Using a separate namespace provides each network with separate resources and allows you to set unique access policies for each network. You need to follow these deployment instructions to deploy a separate operator and console for each project.
{: important}

When you create a new project, a new namespace is created with the same name as your project. You can verify that the existence of the new namespace by using the `oc get namespace` command:
```
$ oc get namespace
NAME                                STATUS    AGE
blockchain-project                  Active    2m
```

You can also use the CLI to find the available storage classes for your namespace. If you created a new storage class for your deployment, that storage class must be visible in the output in the following command:
```
kubectl get storageclasses
```
{:codeblock}

## Create a secret for your entitlement key
{: #deploy-ocp-docker-registry-secret}

You've already created a secret for the entitlement key in the `ibpinfra` namespace or project, now you need to create one in your {{site.data.keyword.blockchainfull_notm}} Platform namespace or project. After you purchase the {{site.data.keyword.blockchainfull_notm}} Platform, you can access the [My IBM dashboard](https://myibm.ibm.com/dashboard/){: external} to obtain your entitlement key for the offering. You need to store the entitlement key on your cluster by creating a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/){: external}. Kubernetes secrets are used to securely store the key on your cluster and pass it to the operator and the console deployments.

Run the following command to create the secret and add it to your namespace or project:
```
kubectl create secret docker-registry docker-key-secret --docker-server=cp.icr.io --docker-username=cp --docker-password=<KEY> --docker-email=<EMAIL> -n <NAMESPACE>
```
{:codeblock}
- Replace `<KEY>` with your entitlement key.
- Replace `<EMAIL>` with your email address.
- Replace `<NAMESPACE>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment namespace or OpenShift project.

The name of the secret that you are creating is `docker-key-secret`. This value is used by the operator to deploy the offering in future steps. If you change the name of any of secrets that you create, you need to change the corresponding name in future steps.
{: note}


## Add security and access policies
{: #deploy-ocp-scc}

The {{site.data.keyword.blockchainfull_notm}} Platform requires specific security and access policies to be added to your project. The contents of a set of `.yaml` files are provided here for you to copy and edit to define the security policies for your project. You must save these files to your local system and then add them your project by using the OpenShift CLI. These steps need to be completed by a cluster administrator. Also, be aware that the peer `init` and `dind` containers that get deployed are required to run in privileged mode.

### Apply the Security Context Constraint

Copy the security context constraint object below and save it to your local system as `ibp-scc.yaml`. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```yaml
allowHostDirVolumePlugin: true
allowHostIPC: true
allowHostNetwork: true
allowHostPID: true
allowHostPorts: true
allowPrivilegeEscalation: true
allowPrivilegedContainer: true
allowedCapabilities:
- NET_BIND_SERVICE
- CHOWN
- DAC_OVERRIDE
- SETGID
- SETUID
- FOWNER
apiVersion: security.openshift.io/v1
defaultAddCapabilities: null
fsGroup:
  type: RunAsAny
groups:
- system:cluster-admins
- system:authenticated
kind: SecurityContextConstraints
metadata:
  name: <PROJECT_NAME>
readOnlyRootFilesystem: false
requiredDropCapabilities: null
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: RunAsAny
supplementalGroups:
  type: RunAsAny
volumes:
- "*"
```
{:codeblock}

After you save and edit the file, run the following commands to add the file to your cluster and add the policy to your project.
```
oc apply -f ibp-scc.yaml -n <PROJECT_NAME>
oc adm policy add-scc-to-user <PROJECT_NAME> system:serviceaccounts:<PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name that you want to use for your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

If the command is successful, you can see a response that is similar to the following example:
```
securitycontextconstraints.security.openshift.io/blockchain-project created
scc "blockchain-project" added to: ["system:serviceaccounts:blockchain-project"]
```

### Apply the ClusterRole

Copy the following text to a file on your local system and save the file as `ibp-clusterrole.yaml`. This file defines the required ClusterRole for the PodSecurityPolicy. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <PROJECT_NAME>
rules:
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - persistentvolumeclaims
  - persistentvolumes
  verbs:
  - '*'
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - 'get'
- apiGroups:
  - "*"
  resources:
  - pods
  - pods/log
  - services
  - endpoints
  - persistentvolumeclaims
  - persistentvolumes
  - events
  - configmaps
  - secrets
  - ingresses
  - roles
  - rolebindings
  - serviceaccounts
  - nodes
  - routes
  - routes/custom-host
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - namespaces
  - nodes
  verbs:
  - get
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  verbs:
  - '*'
- apiGroups:
  - monitoring.coreos.com
  resources:
  - servicemonitors
  verbs:
  - get
  - create
- apiGroups:
  - apps
  resourceNames:
  - ibp-operator
  resources:
  - deployments/finalizers
  verbs:
  - update
- apiGroups:
  - ibp.com
  resources:
  - '*'
  verbs:
  - '*'
- apiGroups:
  - config.openshift.io
  resources:
  - '*'
  verbs:
  - '*'
```
{:codeblock}

After you save and edit the file, run the following commands. Replace `<PROJECT_NAME>` with your project.
```
oc apply -f ibp-clusterrole.yaml -n <PROJECT_NAME>
oc adm policy add-scc-to-group <PROJECT_NAME> system:serviceaccounts:<PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

If successful, you can see a response that is similar to the following example:
```
clusterrole.rbac.authorization.k8s.io/blockchain-project created
scc "blockchain-project" added to groups: ["system:serviceaccounts:blockchain-project"]
```

### Apply the ClusterRoleBinding

Copy the following text to a file on your local system and save the file as `ibp-clusterrolebinding.yaml`. This file defines the ClusterRoleBinding. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: <PROJECT_NAME>
subjects:
- kind: ServiceAccount
  name: default
  namespace: <PROJECT_NAME>
roleRef:
  kind: ClusterRole
  name: <PROJECT_NAME>
  apiGroup: rbac.authorization.k8s.io
```
{:codeblock}

After you save and edit the file, run the following commands:
```
oc apply -f ibp-clusterrolebinding.yaml -n <PROJECT_NAME>
oc adm policy add-cluster-role-to-user <PROJECT_NAME> system:serviceaccounts:<PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

If successful, you can see a response that is similar to the following example:
```
clusterrolebinding.rbac.authorization.k8s.io/blockchain-project created
cluster role "blockchain-project" added: "system:serviceaccounts:blockchain-project"
```


## Deploy the {{site.data.keyword.blockchainfull_notm}} Platform operator
{: #deploy-ocp-operator}

The {{site.data.keyword.blockchainfull_notm}} Platform uses an operator to install the {{site.data.keyword.blockchainfull_notm}} Platform console. You can deploy the operator on your cluster by adding a custom resource to your project by using the OpenShift CLI. The custom resource pulls the operator image from the Docker registry and starts it on your cluster.

Copy the following text to a file on your local system and save the file as `ibp-operator.yaml`.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ibp-operator
  labels:
    release: "operator"
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibp"
    app.kubernetes.io/managed-by: "ibp-operator"
spec:
  replicas: 1
  strategy:
    type: "Recreate"
  selector:
    matchLabels:
      name: ibp-operator
  template:
    metadata:
      labels:
        name: ibp-operator
        release: "operator"
        helm.sh/chart: "ibm-ibp"
        app.kubernetes.io/name: "ibp"
        app.kubernetes.io/instance: "ibp"
        app.kubernetes.io/managed-by: "ibp-operator"  
      annotations:
        productName: "IBM Blockchain Platform"
        productID: "54283fa24f1a4e8589964e6e92626ec4"
        productVersion: "2.5"
        productChargedContainers: ""
        productMetric: "VIRTUAL_PROCESSOR_CORE"
    spec:
      hostIPC: false
      hostNetwork: false
      hostPID: false
      serviceAccountName: default
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 2000
      imagePullSecrets:
        - name: docker-key-secret
      containers:
        - name: ibp-operator
          image: cp.icr.io/cp/ibp-operator:2.5.0-20200618-amd64
          command:
          - ibp-operator
          imagePullPolicy: Always
          securityContext:
            privileged: false
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: false
            runAsNonRoot: false
            runAsUser: 1001
            capabilities:
              drop:
              - ALL
              add:
              - CHOWN
              - FOWNER
          livenessProbe:
            tcpSocket:
              port: 8383
            initialDelaySeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            tcpSocket:
              port: 8383
            initialDelaySeconds: 10
            timeoutSeconds: 5
            periodSeconds: 5
          env:
            - name: WATCH_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: OPERATOR_NAME
              value: "ibp-operator"
            - name: CLUSTERTYPE
              value: OPENSHIFT
          resources:
            requests:
              cpu: 100m
              memory: 200Mi
            limits:
              cpu: 100m
              memory: 200Mi
```
{:codeblock}
- If you changed the name of the Docker key secret, then you need to edit the field of `name: docker-key-secret`.
- If you are using OpenShift Container Platform v4.2 on LinuxONE, you need to make the following additional customizations:
   1. In the `spec.affinity` section, change `amd64` to `s390x`.
   2. In the `spec.containers` section, replace `amd64` in the operator `images` tag with `s390x`.

Then, use the `kubectl` CLI to add the custom resource to your project.

```
kubectl apply -f ibp-operator.yaml -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

You can confirm that the operator deployed by running the command `kubectl get deployment -n <PROJECT_NAME>`. If your operator deployment is successful, then you can see the following tables with four ones displayed. The operator takes about a minute to deploy.
```
NAME           READY     UP-TO-DATE   AVAILABLE   AGE
ibp-operator   1/1       1            1           46s
```

## Deploy the {{site.data.keyword.blockchainfull_notm}} Platform console
{: #deploy-ocp-console}

When the operator is running on your namespace, you can apply a custom resource to start the {{site.data.keyword.blockchainfull_notm}} Platform console on your cluster. You can then access the console from your browser. Note that you can deploy only one console per OpenShift project.

Save the custom resource definition below as `ibp-console.yaml` on your local system. If you changed the name of the entitlement key secret, then you need to edit the field of `name: docker-key-secret`.

```yaml
apiVersion: ibp.com/v1alpha2
kind: IBPConsole
metadata:
  name: ibpconsole
spec:
  arch:
  - amd64
  license: accept
  serviceAccountName: default
  email: "<EMAIL>"
  password: "<PASSWORD>"
  registryURL: cp.icr.io/cp
  imagePullSecrets:
    - docker-key-secret
  networkinfo:
    domain: <DOMAIN>
  storage:
    console:
      class: default
      size: 10Gi
```
{:codeblock}

You need to specify the external endpoint information of the console in the `ibp-console.yaml` file:
- Replace `<DOMAIN>` with the name of your cluster domain. You can find this value by using the OpenShift web console. Use the dropdown menu next to **OpenShift Container Platform** at the top left of the page to switch from **Service Catalog** to **Cluster Console**. Examine the url for that page. It will be similar to `console.xyz.abc.com/k8s/cluster/projects`. The value of the domain then would be `xyz.abc.com`, after removing `console` and `/k8s/cluster/projects`.

You need to provide the user name and password that is used to access the console for the first time:
- Replace `<EMAIL>` with the email address of the console administrator.
- Replace `<PASSWORD>` with the password of your choice. This password also becomes the default password of the console until it is changed.

You also need to make additional edits to the file depending on your choices in the deployment process:
- If you changed the name of your Docker key secret, change corresponding value of the `imagePullSecrets:` field.
- If you created a new storage class for your network, provide the storage class that you created to the `class:` field.

If you are deploying on OpenShift Container Platform v4.2 or v4.3 on LinuxONE, you need to replace:
```yaml
arch:
- amd64
```
in the `spec:` section with:
```yaml
arch:
- s390x
```

If you are running OpenShift on Azure, you also need to change the storage class from `default` to `azure-standard`, unless you created your own storage class.
{: tip}

Because you can only run the following command once, you should review the [Advanced deployment options](#console-deploy-ocp-advanced) in case any of the options are relevant to your configuration, before you install the console.  For example, if you are deploying your console on a multizone cluster, you need to configure that before you run the following step to install the console.
{: important}

After you update the file, you can use the CLI to install the console.
```
kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project. The console can take a few minutes to deploy.

### Advanced deployment options
{: #console-deploy-ocp-advanced}

Before you deploy the console, you can edit the `ibp-console.yaml` file to allocate more resources to your console or use zones for high availability in a multizone cluster. To take advantage of these deployment options, you can use the console resource definition with the `resources:` and `clusterdata:` sections added:

```yaml
apiVersion: ibp.com/v1alpha2
kind: IBPConsole
metadata:
  name: ibpconsole
spec:
  arch:
  - amd64
  license: accept
  serviceAccountName: default
  proxyIP:
  email: "<EMAIL>"
  password: "<PASSWORD>"
  registryURL: cp.icr.io/cp
  imagePullSecrets:
    - docker-key-secret
  networkinfo:
      domain: <DOMAIN>
  storage:
    console:
      class: default
      size: 10Gi
  clusterdata:
    zones:
  resources:
    console:
      requests:
        cpu: 500m
        memory: 1000Mi
      limits:
        cpu: 500m
        memory: 1000Mi
    configtxlator:
      limits:
        cpu: 25m
        memory: 50Mi
      requests:
        cpu: 25m
        memory: 50Mi
    couchdb:
      limits:
        cpu: 500m
        memory: 1000Mi
      requests:
        cpu: 500m
        memory: 1000Mi
    deployer:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
```
{:codeblock}


- You can use the `resources:` section to allocate more resources to your console. The values in the example file are the default values allocated to each container. Allocating more resources to your console allows you to operate a larger number of nodes or channels. You can allocate more resources to a currently running console by editing the resource file and applying it to your cluster. The console will restart and return to its previous state, allowing you to operate all of your exiting nodes and channels.
  ```
  kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
  ```
  {:codeblock}
  Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

- If you plan to use the console with a multizone Kubernetes cluster, you need to add the zones to the `clusterdata.zones:` section of the file. When zones are provided to the deployment, you can select the zone that a node is deployed to using the console or the APIs. As an example, if you are deploying to a cluster across the zones of dal10, dal12, and dal13, you would add the zones to the file by using the format below.
  ```yaml
  clusterdata:
    zones:
      - dal10
      - dal12
      - dal13
  ```
  {:codeblock}

  When you finish editing the file, apply it to your cluster.
  ```
  kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
  ```
  {:codeblock}
  Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

Unlike the resource allocation, you cannot add zones to a running network. If you have already deployed a console and used it to create nodes on your cluster, you will lose your previous work. After the console restarts, you need to deploy new nodes.    
{: Important}

### Use your own TLS Certificates (Optional)

The {{site.data.keyword.blockchainfull_notm}} Platform console uses TLS certificates to secure the communication between the console and your blockchain nodes and between the console and your browser. You have the option of creating your own TLS certificates and providing them to the console by using a Kubernetes secret. If you skip this step, the console creates its own self-signed TLS certificates during deployment.

This step needs to be performed before the console is deployed.
{: important}

You can use a Certificate Authority or tool to create the TLS certificates for the console. The TLS certificate needs to include the hostname of the console and the proxy in the subject name or the alternative domain names. The console and proxy hostname are in the following format:

**Console hostname:** ``<PROJECT_NAME>-ibpconsole-console.<DOMAIN>``  
**Proxy hostname:** ``<PROJECT_NAME>-ibpconsole-proxy.<DOMAIN>``  

- Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.
- Replace `<DOMAIN>` with the name of your cluster domain. You can find this value by using the OpenShift web console. Use the dropdown menu next to **OpenShift Container Platform** at the top of the page to switch from **Service Catalog** to **Cluster Console**. Examine the url for that page. It will be similar to `console.xyz.abc.com/k8s/cluster/projects`. The value of the domain then would be `xyz.abc.com`, after removing `console` and `/k8s/cluster/projects`.

Navigate to the TLS certificates that you plan to use on your local system. Name the TLS certificate `tlscert.pem` and the corresponding private key `tlskey.pem`. Run the following command to create the Kubernetes secret and add it to your OpenShift project. The TLS certificate and key need to be in PEM format.
```
kubectl create secret generic console-tls-secret --from-file=tls.crt=./tlscert.pem --from-file=tls.key=./tlskey.pem -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

After you create the secret, add the `tlsSecretName` field to the `spec:` section of `ibp-console.yaml` with one indent added, at the same level as the `resources:` and `clusterdata:` sections of the advanced deployment options. You must provide the name of the TLS secret that you created to the field. The following example deploys a console with the TLS certificate and key stored in a secret named `"console-tls-secret"`:

```yaml
apiVersion: ibp.com/v1alpha2
kind: IBPConsole
metadata:
  name: ibpconsole
spec:
  arch:
  - amd64
  license: accept
  serviceAccountName: default
  proxyIP:
  email: "<EMAIL>"
  password: "<PASSWORD>"
  registryURL: cp.icr.io/cp
  imagePullSecrets:
    - docker-key-secret
  networkinfo:
      domain: <DOMAIN>
  storage:
    console:
      class: default
      size: 10Gi
  tlsSecretName: "console-tls-secret"
  clusterdata:
    zones:
      - dal10
      - dal12
      - dal13
```
{:codeblock}

When you finish editing the file, you can apply it to your cluster in order to secure communications with your own TLS certificates:
```
kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

### Verifying the console installation

You can confirm that the operator deployed by running the command `kubectl get deployment -n <PROJECT_NAME>`. If your console deployment is successful, you can see `ibpconsole` added to the deployment table, with four ones displayed. The console takes a few minutes to deploy. You might need to click refresh and wait for the table to be updated.
```
NAME           READY     UP-TO-DATE   AVAILABLE   AGE
ibp-operator   1/1       1            1           10m
ibpconsole     1/1       1            1           4m
```

The console consists of four containers that are deployed inside a single pod:
- `optools`: The console UI.
- `deployer`: A tool that allows your console to communicate with your deployments.
- `configtxlator`: A tool used by the console to read and create channel updates.
- `couchdb`: An instance of CouchDB that stores the data from your console, including your authorization information.

If there is an issue with your deployment, you can view the logs from one of the containers inside the pod. First, run the following command to get the name of the console pod:
```
kubectl get pods -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

Then, use the following command to get the logs from one of the four containers listed above:
```
kubectl logs -f <pod_name> <container_name> -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

As an example, a command to get the logs from the UI container would look like the following example:
```
kubectl logs -f ibpconsole-55cf9db6cc-856nz optools -n blockchain-project
```
{:codeblock}

## Log in to the console
{: #deploy-ocp-log-in}

You can use your browser to access the console by browsing to the console URL:

```
https://<PROJECT_NAME>-ibpconsole-console.<DOMAIN>
```
{: codeblock}

- Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.
- Replace `<DOMAIN>` with the name of your cluster domain. You passed this value to the `DOMAIN:` field of the `ibp-console.yaml` file.

Your console URL looks similar to the following example:

```
https://blockchain-project-ibpconsole-console.xyz.abc.com
```
{: codeblock}

You can also find your console URL by logging in to your OpenShift cluster and running the following command:
```
oc get routes -n <PROJECT_NAME>
```
{:codeblock}
Replace `<PROJECT_NAME>` with the name of your {{site.data.keyword.blockchainfull_notm}} Platform deployment project.

In the output of the command, you can see the URLs for the proxy and the console. You need to add `https://` to the beginning console URL to access the console. You do not need to add a port to the URL.

In your browser, you can see the console login screen:
- For the **User ID**, use the value you provided for the `email:` field in the `ibp-console.yaml` file.
- For the **Password**, use the value you encoded for the `password:` field in the `ibp-console.yaml` file. This password becomes the default password for the console that all new users use to log in to the console. After you log in for the first time, you will be asked to provide a new password that you can use to log in to the console.

Ensure that you are not using the ESR version of Firefox. If you are, switch to another browser such as Chrome and log in.
{: important}

The administrator who provisions the console can grant access to other users and restrict the actions they can perform. For more information, see [Managing users from the console](/docs/blockchain-sw-25?topic=blockchain-sw-25-console-icp-manage#console-icp-manage-users).

## Next steps
{: #console-deploy-ocp-next-steps}

When you access your console, you can view the **nodes** tab of your console UI. You can use this screen to deploy components on the cluster where you deployed the console. See the [Build a network tutorial](/docs/blockchain-sw-25?topic=blockchain-sw-25-ibp-console-build-network#ibp-console-build-network) to get started with the console. You can also use this tab to operate nodes that are created on other clouds. For more information, see [Importing nodes](/docs/blockchain-sw-25?topic=blockchain-sw-25-ibp-console-import-nodes#ibp-console-import-nodes).

To learn how to manage the users that can access the console, view the logs of your console and your blockchain components, see [Administering your console](/docs/blockchain-sw-25?topic=blockchain-sw-25-console-icp-manage#console-icp-manage).  

Ready to automate the entire deployment process? Check out the [Ansible Playbook](/docs/blockchain-sw-25?topic=blockchain-sw-25-ansible-install-ibp) that can be used to complete all of the steps  in this topic for you.
