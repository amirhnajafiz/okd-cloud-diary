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

### Image Pull Error

### Crash Loop
