# My Cloud Journey

How I learned to work with cloud and deploy my projects.

## Start
I start by installing **OpenShift**.

Built on Red Hat® Enterprise Linux® and Kubernetes, OpenShift Container Platform provides a secure and scalable multi-tenant operating system for today’s enterprise-class applications. It also provides integrated application runtimes and libraries.

You can download _oc_ from this [link](https://access.redhat.com/downloads/content/290/ver=4.10/rhel---8/4.10.9/x86_64/product-software).

After you installed _oc_, test it with the following command:
```shell
oc --help
```

We are going to use **OpenShift** to connect to our Kubernetes cluster and deploy our project.

## Login
In this part, we are going to login in to our Kuber cluster.

Use the following command:
```shell
oc login
```

Now enter your cluster information, for example:
```shell
OpenShift server [https://localhost:8443]: https://openshift.example.com 

Username: alice 
Authentication required for https://openshift.example.com (openshift)
Password: ******
Login successful. 

You do not have any projects. You can try to create a new project, by running

    oc new-project <projectname> 

Welcome to OpenShift! See 'oc help' to get started.
```

Now you are connected to your cluster.

## Using helm
After we connect to our cluster, we install our application using helm:
```shell
helm install RELEASE CHARTS_DIR
```

This should start your application on Kubernetes cluster.

If you want to uninstall the application, use the following command:
```shell
helm uninstall RELEASE
```

## Errors
### Role binding Error
My first error in the cloud was Role binding error. This error comes from **service account**. A service account provides an identity for processes that run in a Pod.

When I was deploying my project i got the following error:
```shell
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [unable to recognize "": no matches for kind "Role" in 
version "rbac.authorization.k8s.io/v1beta1", unable to recognize "": no matches for kind "RoleBinding" 
in version "rbac.authorization.k8s.io/v1beta1"]
```

I manage to fix this by removing all of the code lines that were referred to **service account**.

### Dependencies Error
For my project, I had two other dependencies, Jaeger and MQX. So when you want to add your dependencies to your project charts
all you have to do is to find that dependencies charts and import them in **Chart.yaml** like this:
```yaml
# application dependencies
dependencies:
  - name: jaeger
    version: 0.56.1
    repository: https://jaegertracing.github.io/helm-charts
```

After that, you need to update your dependencies with the following command:
```shell
helm dep update
```

Now you have the **charts** directory with your dependencies charts. You will also have **Chart.lock** file.

When you want to install your charts into the Kubernetes cluster, you may find some errors that are not in your charts.
These errors usually get fixed by changing your dependencies versions.

For changing your dependencies versions, change the version in **Chart.yaml** file and run the following command again:
```shell
helm dep update
```

If you already deployed your application, you need to run the following command too:
```shell
helm upgrade RELEASE
```

### Image Pull Error
When a Kubernetes cluster creates a new deployment, or updates an existing deployment, it typically needs to pull an image. This is done by the kubelet process on each worker node. For the kubelet to successfully pull the images, they need to be accessible from all nodes in the cluster that match the scheduling request.

The ImagePullBackOff error occurs when the image path is incorrect, the network fails, or the kubelet does not succeed in authenticating with the container registry. Kubernetes initially throws the ErrImagePull error, and then after retrying a few times, “pulls back” and schedules another download attempt. For each unsuccessful attempt, the delay increases exponentially, up to a maximum of 5 minutes.

To identify the ImagePullBackOff error: run the kubectl get pods command. The pod status will show the error like so:
```shell
NAME        READY    STATUS              RESTARTS    AGE
my-pod-1    0/1      ImagePullBackOff    0           2m34s
```

### Crash Loop
