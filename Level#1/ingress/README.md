## 1. Create a Namespace, a SA, the Default Secret, the Customization Config Map, and Custom Resource Definitions

1. Create a namespace and a service account for the Ingress controller:
    ```
    kubectl apply -f common/ns-and-sa.yaml
    ```

1. Create a secret with a TLS certificate and a key for the default server in NGINX:
    ```
    $ kubectl apply -f common/default-server-secret.yaml
    ```


1. Create a config map for customizing NGINX configuration (read more about customization [here](configmap-and-annotations.md)):
    ```
    $ kubectl apply -f common/nginx-config.yaml
    ```

1. (Optional) To use the [VirtualServer and VirtualServerRoute](virtualserver-and-virtualserverroute.md) resources, create the corresponding resource definitions:
    ```
    $ kubectl apply -f common/custom-resource-definitions.yaml
    ```
    Note: in Step 3, make sure the Ingress controller starts with the `-enable-custom-resources` [command-line argument](cli-arguments.md).

## 2. Configure RBAC

If RBAC is enabled in your cluster, create a cluster role and bind it to the service account, created in Step 1:
```
$ kubectl apply -f rbac/rbac.yaml
```

**Note**: To perform this step you must be a cluster admin. Follow the documentation of your Kubernetes platform to configure the admin access. For GKE, see the [Role-Based Access Control](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control) doc.

## 3. Deploy the Ingress Controller

We include two options for deploying the Ingress controller:
* *Deployment*. Use a Deployment if you plan to dynamically change the number of Ingress controller replicas.
* *DaemonSet*. Use a DaemonSet for deploying the Ingress controller on every node or a subset of nodes.

### 3.1 Create a Deployment

For NGINX, run:
```
$ kubectl apply -f deployment/nginx-ingress.yaml
```

For NGINX Plus, run:
```
$ kubectl apply -f deployment/nginx-plus-ingress.yaml
```

**Note**: Update the `nginx-plus-ingress.yaml` with the container image that you have built.

Kubernetes will create one Ingress controller pod.


### 3.2 Create a DaemonSet

For NGINX, run:
```
$ kubectl apply -f daemon-set/nginx-ingress.yaml
```

For NGINX Plus, run:
```
$ kubectl apply -f daemon-set/nginx-plus-ingress.yaml
```

**Note**: Update the `nginx-plus-ingress.yaml` with the container image that you have built.

Kubernetes will create an Ingress controller pod on every node of the cluster. Read [this doc](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) to learn how to run the Ingress controller on a subset of nodes, instead of every node of the cluster.

### 3.3 Check that the Ingress Controller is Running

Run the following command to make sure that the Ingress controller pods are running:
```
$ kubectl get pods --namespace=nginx-ingress
```

## 4. Get Access to the Ingress Controller

**If you created a daemonset**, ports 80 and 443 of the Ingress controller container are mapped to the same ports of the node where the container is running. To access the Ingress controller, use those ports and an IP address of any node of the cluster where the Ingress controller is running.

**If you created a deployment**, below are two options for accessing the Ingress controller pods.

### 4.1 Service with the Type NodePort

Create a service with the type *NodePort*:
```
$ kubectl create -f service/nodeport.yaml
```
Kubernetes will randomly allocate two ports on every node of the cluster. To access the Ingress controller, use an IP address of any node of the cluster along with the two allocated ports. Read more about the type NodePort [here](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport).

### 4.2 Service with the Type LoadBalancer

Create a service with the type *LoadBalancer*. Kubernetes will allocate and configure a cloud load balancer for load balancing the Ingress controller pods.

Create a service using a manifest for your cloud provider:
* For GCP or Azure, run:
    ```
    $ kubectl apply -f service/loadbalancer.yaml
    ```
* For AWS, run:
    ```
    $ kubectl apply -f service/loadbalancer-aws-elb.yaml
    ```
   

