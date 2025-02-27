# Helm Chart
The Helm chart for this driver includes the following features:

The Helm chart for this driver includes the following features:

- A Deployment to create pods for your driver application
- A Service to expose your application internally and externally (with a NodePort)
- Ingress rule to expose your application externally with Ingress (Ingress controller required on Kubernetes cluster. By default, LM installs an Ingress controller for you)
- Configurable options for:
    - Location and version of Docker image
    - Number of replicas of the driver
    - Property groups configured on your application (i.e. override the properties in the default_config file)
    - Log level
    - uWSGI container used by your application
    - Pod affinity/anti-affinity
    - Node affinity/anti/affinity
    - NodePort
    - Ingress Host
- Necessary Deployment labels so logs from the driver may be viewed in the dashboard provided with LM (Kibana)

## Building helm chart

```
1. cd helm
```

2. Run the following command to build the helm chart

```
   tar -czvf sol005-lifecycle-driver.tgz *
```

## Deploy Helm Chart

To deploy the Helm chart will need Helm installed and initialised against a Kubernetes cluster (e.g. `helm init` on your Kubernetes cluster master node).

Install the chart with the default configuration using the install command:

```
helm install --name sol005-lifecycle-driver <path to chart>
```
   
Configuration for the Helm deployment can be provided with a **Helm values** file on the `-f` option.

```
helm install --name sol005-lifecycle-driver <path to chart> -f <path to Helm values file>
```

By default, the Deployment included in this chart will expect the driver docker image to be available on the Kubernetes worker node (e.g. the image can be seen in the list returned by `docker images`). If the image is not on the node, you should [transfer the image to the node](#transfer-docker-image-to-node) or [use a docker registry](#use-docker-registry).

To override configuration for your driver application (e.g. Kafka address, other property groups configured by the user), add the following to a custom **Helm values file**:

```
app:
  override:
    ## Add the properties to configure as YAML here
    propA: valueA 
```

Once you have installed the chart you can check the status of your deployment with:

```
kubectl get pods
```

Or:

```
helm status sol005-lifecycle-driver
```

## Transfer Docker Image to Node

To transfer it to the node you can use `docker save` locally, then `docker load` on the node:

Locally:
```
docker save -o sol005-lifecycle-driver-img.tar sol005-lifecycle-driver:<release version number>
```

Transfer the file using scp:
```
scp sol005-lifecycle-driver-img.tar <user>@<worker-node-ip>:/home/<user>
```

On Node:
```
docker load -i /home/<user>/sol005-lifecycle-driver-img.tar
```  

## Use Docker Registry

In a custom Helm values file add the following:

```
docker:
  image: <registry host and port>/sol005-lifecycle-driver
  version: <image version>
  imagePullPolicy: IfNotPresent
```

Include this values file on the Helm install command with the `-f` option.

Note: if the Docker registry is insecure you need to inform the docker daemon (usually by adding it to a "insecure-registries" in `/etc/docker/daemon.json`).
