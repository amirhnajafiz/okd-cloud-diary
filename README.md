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
A CrashloopBackOff means that you have a pod starting, crashing, starting again, and then crashing again.

A PodSpec has a restartPolicy field with possible values Always, OnFailure, and Never which applies to all containers in a pod. The default value is Always and the restartPolicy only refers to restarts of the containers by the kubelet on the same node (so the restart count will reset if the pod is rescheduled in a different node). Failed containers that are restarted by the kubelet are restarted with an exponential back-off delay (10s, 20s, 40s …) capped at five minutes, and is reset after ten minutes of successful execution. This is an example of a PodSpec with the restartPolicy field:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dummy-pod
spec:
  containers:
    - name: dummy-pod
      image: ubuntu
  restartPolicy: Always
```

A quick Google search will show us that crash loop events can happen for a number of different reasons (and they happen frequently). Here are some of the umbrella causes for why they occur:

- The application inside the container keeps crashing. Here, we can highlight several common situations:
    - Error in the application configuration. A wrong value or format can make the application exit just after start.
    - Bugs or not caught exceptions.
    - One of the downstream services on which the application relies can’t be reached or the connection fails (database, backend, etc.).
- Errors in the manifest or pod configuration, such as:
    - Trying to bind an already used port.
    - Wrong command arguments for the container.
    - Errors in liveness probes.
    - Read-only filesystem.

Run your standard kubectl get pods command and you’ll be able to see the status of any pod that is currently in CrashLoopBackOff:
```shell
kubectl get pods --namespace nginx-crashloop
NAME                     READY     STATUS             RESTARTS   AGE
flask-7996469c47-d7zl2   1/1       Running            1          77d
flask-7996469c47-tdr2n   1/1       Running            0          77d
nginx-5796d5bc7c-2jdr5   0/1       CrashLoopBackOff   2          1m
nginx-5796d5bc7c-xsl6p   0/1       CrashLoopBackOff   2          1m
```

You can manually trigger a Sysdig capture at any point in time by selecting the host where you see the CrashLoopBackOff is occurring and starting the capture. You can take it manually with Sysdig open source if you have it installed on that host. But here will take advantage of the Sysdig Monitor capabilities that can automatically take this capture file as a response to an alert, in this case a CrashLoopBackOff alert.

## Resource Management
Based on what you want to deploy, you should always manage your resources. I once deployed a project without noticing 
that I have few memory and cpu resource allocated for this application, and I go error, my app deployed but the pods 
did not run.

You can set your resources in your values.yaml file:
```yaml
resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
```

## Different kind of deployments
