
---

copyright:
  years: 2018, 2019
lastupdated: "2019-09-24"

keywords: OpenShift, IBM Blockchain Platform console, deploy, resource requirements, storage, parameters

subcollection: blockchain-rhos

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:note: .note}
{:important: .important}
{:tip: .tip}
{:pre: .pre}

# Deploying {{site.data.keyword.blockchainfull_notm}} Platform v2.1.0 from IBM
{: #deploy-ocp-ibm}

These instructions are provided for IBMers only to access the images.

## Log in to your OpenShift cluster
{: #deploy-ocp-ibm-login-firewall}

Before you can complete the next steps, you need to log in to your cluster by using the OpenShift CLI. You can log in to your cluster by using the OpenShift web console.

1. Open the OpenShift web console. In the upper right corner of the cluster overview page, click **OpenShift web console**.

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

## Create a new project
{: #deploy-ocp-ibm-project-firewall}

After you connect to your cluster, create a new project for your deployment of {{site.data.keyword.blockchainfull_notm}} Platform. You can create a new project by using the OpenShift web console or OpenShift CLI. The new project needs to be created by a cluster administrator.

If you are using the CLI, create a new project by the following command:
```
oc new-project <PROJECT_NAME>
```
{:codeblock}

Replace `<PROJECT_NAME>` with the name of your project.

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
oc get storageclasses
```
{:codeblock}

## Add security and access policies
{: #deploy-ocp-ibm-scc-firewall}

The {{site.data.keyword.blockchainfull_notm}} Platform requires specific security and access policies to be added to your project. The contents of a set of `.yaml` files are provided here for you to copy and edit to define the security policies for your project. You must save these files to your local system and then add them your project by using the OpenShift CLI. These steps need to be completed by a cluster administrator. Also, be aware that the peer `init` and `dind` containers that get deployed are required to run in privileged mode.

### Apply the Security Context Constraint

Copy the security context constraint object below and save it to your local system as `ibp-scc.yaml`. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```
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

After you save and edit the file, run the following commands to add the file to your cluster and add the policy to your project. Replace `<PROJECT_NAME>` with your project.
```
oc apply -f ibp-scc.yaml -n <PROJECT_NAME>
oc adm policy add-scc-to-user <PROJECT_NAME> system:serviceaccounts:<PROJECT_NAME>
```
{:codeblock}

If the command is successful, you can see a response that is similar to the following example:
```
securitycontextconstraints.security.openshift.io/blockchain-project created
scc "blockchain-project" added to: ["system:serviceaccounts:blockchain-project"]
```

### Apply the ClusterRole

Copy the following text to a file on your local system and save the file as `ibp-clusterrole.yaml`. This file defines the required ClusterRole for the PodSecurityPolicy. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: <PROJECT_NAME>
rules:
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - persistentvolumeclaims
  - persistentvolumes
  - customresourcedefinitions
  verbs:
  - '*'
- apiGroups:
  - "*"
  resources:
  - pods
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
  - ibpservices
  - ibpcas
  - ibppeers
  - ibpfabproxies
  - ibporderers
  verbs:
  - '*'
- apiGroups:
  - ibp.com
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

If successful, you can see a response that is similar to the following example:
```
clusterrole.rbac.authorization.k8s.io/blockchain-project created
scc "blockchain-project" added to groups: ["system:serviceaccounts:blockchain-project"]
```

### Apply the ClusterRoleBinding

Copy the following text to a file on your local system and save the file as `ibp-clusterrolebinding.yaml`. This file defines the ClusterRoleBinding. Edit the file and replace `<PROJECT_NAME>` with the name of your project.

```
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

After you save and edit the file, run the following commands. Replace `<PROJECT_NAME>` with your project.
```
oc apply -f ibp-clusterrolebinding.yaml -n <PROJECT_NAME>
oc adm policy add-cluster-role-to-user <PROJECT_NAME> system:serviceaccounts:<PROJECT_NAME>
```
{:codeblock}

If successful, you can see a response that is similar to the following example:
```
clusterrolebinding.rbac.authorization.k8s.io/blockchain-project created
cluster role "blockchain-project" added: "system:serviceaccounts:blockchain-project"
```

## Create a secret for your entitlement key
{: #deploy-ocp-ibm-docker-registry-secret-firewall}

After you push the {{site.data.keyword.blockchainfull_notm}} Platform images to your own docker registry, you need to store the password to that registry on your cluster by creating a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/){: external}. Using a Kubernetes secret allows you to securely store the key on your cluster and pass it to the operator and the console deployments.

Run the following command to create the secret and add it to your OpenShift Project:
```
kubectl create secret docker-registry docker-key-secret --docker-server=<LOCAL_REGISTRY> --docker-username=<USER> --docker-password=<LOCAL_REGISTRY_PASSWORD> --docker-email=<EMAIL> -n <PROJECT_NAME>
```
{:codeblock}

- Replace `<USER>` with your user name
- Replace `<EMAIL>` with your email address.
- Replace `<LOCAL_REGISTRY_PASSWORD>` with the password to your registry.
- Replace `<LOCAL_REGISTRY>` with the url of your local registry.

The name of the secret that you are creating is `docker-key-secret`. This value is used by the operator to deploy the offering in future steps. If you change the name of any of secrets that you create, you need to change the corresponding name in future steps.
{: note}

## Deploy the {{site.data.keyword.blockchainfull_notm}} Platform operator
{: #deploy-ocp-ibm-operator}

The {{site.data.keyword.blockchainfull_notm}} Platform uses an operator to install the {{site.data.keyword.blockchainfull_notm}} Platform console. You can deploy the operator on your cluster by adding a custom resource to your project by using the OpenShift CLI. The custom resource pulls the operator image from the Docker registry and starts it on your cluster.

Copy the following text to a file on your local system and save the file as `ibp-operator.yaml`. Replace `<LOCAL_REGISTRY>` with the url of your local registry. If you changed the name of the Docker key secret, then you need to edit the field of `name: docker-key-secret`.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ibp-operator
  labels:
    release: "operator"
    helm.sh/chart: "ibm-ibp"
    app.kubernetes.io/name: "ibp"
    app.kubernetes.io/instance: "ibpoperator"
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
        app.kubernetes.io/instance: "ibpoperator"
        app.kubernetes.io/managed-by: "ibp-operator"
      annotations:
        productName: "IBM Blockchain Platform"
        productID: "54283fa24f1a4e8589964e6e92626ec4"
        productVersion: "2.1.0"
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
      imagePullSecrets:
        - name: docker-key-secret
      containers:
        - name: ibp-operator
          image: <LOCAL_REGISTRY>/ibp-operator:2.1.0-20190924-amd64
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
            - name: ISOPENSHIFT
              value: "true"
          resources:
            requests:
              cpu: 100m
              memory: 200Mi
            limits:
              cpu: 100m
              memory: 200Mi
```
{:codeblock}

Then, use the `kubectl` CLI to add the custom resource to your project.

```
kubectl apply -f ibp-operator.yaml -n <PROJECT_NAME>
```
{:codeblock}

You can confirm that the operator deployed by running the command `kubectl get deployment -n <PROJECT_NAME>`. If your operator deployment is successful, then you can see the following tables with four ones displayed. The operator takes about a minute to deploy.
```
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ibp-operator   1         1         1            1           1m
```

## Deploy the {{site.data.keyword.blockchainfull_notm}} Platform console
{: #deploy-ocp-ibm-console}

When the operator is running on your namespace, you can apply a custom resource to start the {{site.data.keyword.blockchainfull_notm}} Platform console on your cluster. You can then access the console from your browser. Note that you can deploy only one console per OpenShift project.

Save the custom resource definition below as `ibp-console.yaml` on your local system. If you changed the name of the entitlement key secret, then you need to edit the field of `name: docker-key-secret`.

```
apiVersion: ibp.com/v1alpha1
kind: IBPConsole
metadata:
  name: ibpconsole
spec:
  license: accept
  serviceAccountName: default
  email: "<EMAIL>"
  password: "<PASSWORD>"
  image:
      imagePullSecret: docker-key-secret
      consoleInitImage: <LOCAL_REGISTRY>/ibp-init
      consoleInitTag: 2.1.0-20190924-amd64
      consoleImage: <LOCAL_REGISTRY>/ibp-console
      consoleTag: 2.1.0-20190924-amd64
      configtxlatorImage: <LOCAL_REGISTRY>/ibp-utilities
      configtxlatorTag: 1.4.3-20190924-amd64
      couchdbImage: <LOCAL_REGISTRY>/ibp-couchdb
      couchdbTag: 2.3.1-20190924-amd64
      deployerImage: <LOCAL_REGISTRY>/ibp-deployer
      deployerTag: 2.1.0-20190924-amd64
  versions:
      ca:
        1.4.3-0:
          default: true
          version: 1.4.3-0
          image:
            caInitImage: <LOCAL_REGISTRY>/ibp-ca-init
            caInitTag: 2.1.0-20190924-amd64
            caImage: <LOCAL_REGISTRY>/ibp-ca
            caTag: 1.4.3-20190924-amd64
      peer:
        1.4.3-0:
          default: true
          version: 1.4.3-0
          image:
            peerInitImage: <LOCAL_REGISTRY>/ibp-init
            peerInitTag: 2.1.0-20190924-amd64
            peerImage: <LOCAL_REGISTRY>/ibp-peer
            peerTag: 1.4.3-20190924-amd64
            dindImage: <LOCAL_REGISTRY>/ibp-dind
            dindTag: 1.4.3-20190924-amd64
            fluentdImage: <LOCAL_REGISTRY>/ibp-fluentd
            fluentdTag: 2.1.0-20190924-amd64
            grpcwebImage: <LOCAL_REGISTRY>/ibp-grpcweb
            grpcwebTag: 2.1.0-20190924-amd64
            couchdbImage: <LOCAL_REGISTRY>/ibp-couchdb
            couchdbTag: 2.3.1-20190924-amd64
      orderer:
        1.4.3-0:
          default: true
          version: 1.4.3-0
          image:
            ordererInitImage: <LOCAL_REGISTRY>/ibp-init
            ordererInitTag: 2.1.0-20190924-amd64
            ordererImage: <LOCAL_REGISTRY>/ibp-orderer
            ordererTag: 1.4.3-20190924-amd64
            grpcwebImage: <LOCAL_REGISTRY>/ibp-grpcweb
            grpcwebTag: 2.1.0-20190924-amd64
  networkinfo:
    domain: <DOMAIN>
  storage:
    console:
      class: default
      size: 10Gi
```
{:codeblock}

You need to specify the external endpoint information of the console in the `ibp-console.yaml` file:
- Replace `<LOCAL_REGISTRY>` with the url of your local registry.
- Replace `<DOMAIN>` with the name of your cluster domain. You can find this value by using the OpenShift web console. Use the dropdown menu next to **OpenShift Container Platform** at the top of the page to switch from **Service Catalog** to **Cluster Console**. Examine the url for that page. It will be similar to `console.xyz.abc.com/k8s/cluster/projects`. The value of the domain then would be `xyz.abc.com`, after removing `console` and `/k8s/cluster/projects`.

You need to provide the user name and password that is used to access the console for the first time:
- Replace `<EMAIL>` with the email address of the console administrator.
- Replace `<PASSWORD>` with the password of your choice. This password also becomes the default password of the console until it is changed.

You also need to make additional edits to the file depending on your choices in the deployment process:
- If you changed the name of your Docker key secret, change corresponding value of the `imagePullSecret:` field.
- If you created a new storage class for your network, provide the storage class that you created to the `class:` field.

If you are deploying your console on a multizone cluster, go to the [advanced deployment options](#console-deploy-ocp-advanced) before you deploy the console.
{: important}

After you update the file, you can use the CLI to install the console.
```
kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
```
{:codeblock}

Replace `<PROJECT_NAME>` with the name of your project. Before you install the console, you might want to review the advanced deployment options in the next section. The console can take a few minutes to deploy.

### Advanced deployment options
{: #console-deploy-ocp-advanced}

You can add fields to the `ibp-console.yaml` file to customize the deployment of your console. You can use the additional deployment options to allocate more resources to your cluster, use zones for high availability in a multizone cluster, or provide your own TLS certificates to the console. The new fields must be added to the `spec:` section of `ibp-console.yaml` with one indent added. For example, if you wanted to add the field `newField: newValue` to `ibp-console.yaml`, your file would resemble the following example:
```
apiVersion: ibp.com/v1alpha1
kind: IBPConsole
metadata:
  name: ibpconsole
  spec:
    license: accept
    serviceAccountName: default
    proxyIP:
    email: "<EMAIL>"
    password: "<PASSWORD>"
    networkinfo:
        domain: <DOMAIN>
    storage:
      console:
        class: default
        size: 10Gi
    newField: newValue
```

- You need to add the following fields to `ibp-console.yaml` to allocate more resources to your console. Allocating more resources to your console allows you to operate a larger number of nodes or channels.
  ```
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

  When you finish editing the file, you can apply it to your cluster to allocate more resources to a currently running console. The console will restart and return to its previous state, allowing you to operate all of your exiting nodes and channels.
  ```
  kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
  ```
  {:codeblock}

- If you have labeled the different worker nodes of your cluster with zones, you need to add the zones to the `ibp-console.yaml` file. When zones are provided to the deployment, you can select the zone that a node is deployed to using the console or the APIs. For example, if you are deploying a cluster across the zones of dal10, dal12, and dal13, you would add the following to fields to `ibp-console.yaml`.
  ```
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

If you have already deployed a console and used it to create nodes on your cluster, you will lose your previous work. After the console restarts, you need to deploy new nodes.    
{: Important}

### Use your own TLS Certificates (Optional)

The {{site.data.keyword.blockchainfull_notm}} Platform console uses TLS certificates to secure the communication between the console and your blockchain nodes and between the console and your browser. You have the option of creating your own TLS certificates and providing them to the console by using creating a Kubernetes secret. If you skip this step, the console creates its own self-signed TLS certificates during deployment.

You can use a Certificate Authority or tool to create the TLS certificates for the console. The TLS certificate needs to include the hostname of the console and the proxy in the subject name or the alternative domain names. The console and proxy hostname are in the following format:

**Console hostname:** ``<PROJECT_NAME>-ibpconsole-console.<DOMAIN>``  
**Proxy hostname:** ``<PROJECT_NAME>-ibpconsole-proxy.<DOMAIN>``

- Replace `<PROJECT_NAME>` with the name of the OpenShift project that you created.
- Replace `<DOMAIN>` with the name of your cluster domain. You can find this value by using the OpenShift web console. Use the dropdown menu next to **OpenShift Container Platform** at the top of the page to switch from **Service Catalog** to **Cluster Console**. Examine the url for that page. It will be similar to `console.xyz.abc.com/k8s/cluster/projects`. The value of the domain then would be `xyz.abc.com`, after removing `console` and `/k8s/cluster/projects`.

Navigate to the TLS certificates that you plan to use on your local system. Name the TLS certificate `tlscert.pem` and the corresponding private key `tlskey.pem`. Run the following command to create the Kubernetes secret and add it to your OpenShift project. The TLS certificate and key need to be in PEM format.
```
kubectl create secret generic console-tls-secret --from-file=tls.crt=./tlscert.pem --from-file=tls.key=./tlskey.pem -n <PROJECT_NAME>
```
{:codeblock}

After you create the secret, add the following field to the `spec:` section of `ibp-console.yaml` with one indent added. You must provide name of the TLS secret that you created to the field:
```
tlsSecretName: console-tls-secret
```
{:codeblock}

When you finish editing the file, you can apply it to your cluster to provide new TLS certificates to a deployed console. After the console restarts, the UI returns to its previous state, allowing you to operate all of your exiting nodes and channels.
```
kubectl apply -f ibp-console.yaml -n <PROJECT_NAME>
```
{:codeblock}

### Verifying the console installation

You can confirm that the operator deployed by running the command `kubectl get deployment -n <PROJECT_NAME>`. If your console deployment is successful, you can see `ibpconsole` added to the deployment table, with four ones displayed. The console takes a few minutes to deploy. You might need to click refresh and wait for the table to be updated.
```
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ibp-operator   1         1         1            1           10m
ibpconsole     1         1         1            1           4m
```

The console consists of four containers that are deployed inside a single pod:
- `optools`: The console UI.
- `deployer`: A tool that allows your console to communicate with your deployments.
- `configtxlator`: A tool used by the console to read and create channel updates.
- `couchdb`: An instance of CouchDB that stores the data from your console, including your authorization information.

If there is an issue with your deployment, you can view the logs from an individual container. First, run the following command to get the name of the console pod:
```
kubectl get pods -n <PROJECT_NAME>
```
{:codeblock}

Then, use the following command to get the logs from one of the four containers inside the pod:
```
kubectl logs -f <pod_name> <container_name> -n <PROJECT_NAME>
```
{:codeblock}
As an example, a command to get the logs from the UI container would look like the following example:
```
kubectl logs -f ibpconsole-55cf9db6cc-856nz console -n blockchain-project
```
{:codeblock}

## Log in to the console
{: #deploy-ocp-ibm-log-in}

You can use your browser to access the console by browsing to the console URL:

```
https://<PROJECT_NAME>-ibpconsole-console.<DOMAIN>:443
```

- Replace `<PROJECT_NAME>` with the name of the OpenShift project that you created.
- Replace `<DOMAIN>` with the name of your cluster domain. You passed this value to the `DOMAIN:` field of the `ibp-console.yaml` file.

Your console URL looks similar to the following example:
```
https://blockchain-project-ibpconsole-console.xyz.abc.com:443
```

You can also find your console URL and your proxy URL by using the OpenShift web console. Use the dropdown menu next to **OpenShift Container Platform** at the top of the page to switch from **Service Catalog** to **Cluster Console**. In the left navigation pane, click **Networking** and then **Routes**. Use the **Projects:** dropdown to select all projects. On the page that is displayed, you can see the URLs for the proxy and the console.

When you go to your console URL, your browser will display a screen that states **Your connection is not secure** or **Your connection is not private**. This is because your browser needs to accept the self-signed certificates that are generated by the console. Use the advanced options to make an exception and proceed to the URL. When you see the login screen, open a new tab in your browser and navigate to the proxy URL: `https://<PROJECT_NAME>-ibpconsole-proxy.<DOMAIN>:443`. You need to accept the certificate from this url to communicate with your nodes from your console.
{: important}

On the console login screen, you need to provide the user name and password that is used to access the console for the first time:
- Replace `<EMAIL>` with the email address of the console administrator.
- Replace `<PASSWORD>` with the password of your choice. This password also becomes the default password of the console until it is changed.

Ensure that you are not using the ESR version of Firefox. If you are, switch to another browser such as Chrome and log in.
{: important}

In your browser, you can see the console log in screen:
- For the **User ID**, use the value you provided for the `email:` field in the `ibp-console.yaml` file.
- For the **Password**, use the value you encoded for the `password:` field in the `ibp-console.yaml` file. This password becomes the default password for the console that all new users use to log in to the console. After you log in for the first time, you will be asked to provide a new password that you can use to log in to the console.

The administrator who provisions the console can grant access to other users and restrict the actions they can perform. For more information, see [Managing users from the console](/docs/services/blockchain-rhos?topic=blockchain-rhos-console-icp-manage#console-icp-manage-users).

## Next steps
{: #console-deploy-ocp-next-steps}

When you access your console, you can view the **nodes** tab of your console UI. You can use this screen to deploy components on the cluster where you deployed the console. See the [Build a network tutorial](/docs/services/blockchain-rhos?topic=blockchain-rhos-ibp-console-build-network#ibp-console-build-network) to get started with the console. You can also use this tab to operate nodes that are created on other clouds. For more information, see [Importing nodes](/docs/services/blockchain-rhos?topic=blockchain-rhos-ibp-console-import-nodes#ibp-console-import-nodes).

To learn how to manage the users that can access the console, view the logs of your console and your blockchain components, see [Administering your console](/docs/services/blockchain-rhos?topic=blockchain-rhos-console-icp-manage#console-icp-manage).


