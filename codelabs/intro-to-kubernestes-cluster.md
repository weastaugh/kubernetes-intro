author: Line Moseng and Ingrid Guren
id: lidev-introduction-to-kubernetes-cluster


# Installation and setup 

## Kubernetes cluster
We need a cluster in order to continue with this workshop.
There are two alternatives here.
We recommend the first alternative.
With the second you must skip the tasks to create build triggers and apply changes to the application code.

### Choose between one of these setups: 
1. **Set up your own cluster on Google Cloud Platform**.
You will have to register your credit card, but you will not be charged anything for this workshop.
Complete the steps in [./gcp-setup.md](1a-gcp-setup.mdpp)

2. **Use a service account to authenticate** against a cluster we have already created.
Complete the steps in [./1b-service-account-setup.md](1b-service-account-setup.mdpp)

## Install the Kubernetes command-line tool
1. To operate our cluster, we will use the Kubernetes command line tool, *kubectl*:
  ```
   gcloud components install kubectl
  ```

The cloud SDK installs the tool for you. This tool is not Google Cloud specific, but is used to operate Kubernetes clusters regardless of where they are hosted.

2. You can see if your cluster is created by this command:
   
   ```
   gcloud container clusters list
   ```

    If the status of your cluster is `RUNNING`, move on to the step 3. If there is no output, you might have the wrong project set in your config file. Do this to set the correct project:
  
    - Go back to your browser and click on the dropdown next to `Google Cloud Platform`. This should open a modal where at least one project is listed.
    - Copy the ID of the active project
    - Type this in your terminal:
  
        ```
        gcloud config set project INSERT_PROJECT_ID
        ```
    
    - Run:
    
        ```
        gcloud container clusters list
        ```
    
3. We want to set the default zone of our application, this tells google cloud where to look for the cluster.
We created our cluster in *europe-north1-a* and will set our default zone to this. 

    ```
    gcloud config set compute/zone europe-north1-a
    ``` 

4. The next step is to make sure that the Kubernetes command line tool is authenticated against our new cluster. This is easily done by this neat gcloud command:
    ```
   gcloud container clusters get-credentials cv-cluster
   ```

What this does is to write credentials to the file `~/.kube/config`. You can take a look at that file too see what is written to it.

**Extra task:** If you want bash autocompletion for kubectl, follow [these steps](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion).

## Describe components of the cluster
Now that we are authenticated, we can look at the components in our cluster by using the kubectl command.

1. Remember how Kubernetes consists of nodes? List them by this command:

    ```
   kubectl get nodes
   ```

2. If you want you can get more details about them by describing one of them:

    ```
   kubectl describe nodes <INSERT_NODE_NAME>
   ```

3. We also have different namespaces:

    ```
   kubectl get namespace
   ```

This should list the namespaces `kube-system`, `kube-public` and `default`. The namespace `default`is where we will deploy our applications. `kube-system` is used by Kubernetes, `kube-public` is for resources that does not need authentication and `default` is, as the name says, the default namespace for resources. You can create your own namespaces, e.g. for test and prod.

## Change namespace  
**Only if you did step 2. in the first setup task, and did not set up your own Google Cloud project and cluster**

*The output from the previous command listed more than the default 3 namespaces. This is because we have created one namespace for each service acccount in this workshop. We have given you read/write rights to your own namespace and read rights to the others. This we have done using RBAC (role-based access control) which is a built in feature in Kubernetes. This is not covered in this workshop, but feel free to ask us about it or explore it on your own.*

1. Change namespace to the namespace provided in the email you got:

  ```
  kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
  # Validate it
  kubectl config view | grep namespace: 
  ```

  The second line should output your namespace.




# Google Cloud Platform setup
Follow these steps to set up a cluster in Google Kubernetes Engine.

## Sign up
Create an account on Google Cloud Platform. 
  1. Go to: https://console.cloud.google.com 
  2. Sign up. You will have to register a payment method to complete the sign up. The first 12 months are free, as long as you don't use more than the included $300 credits, so you should not be charged anything for this workshop.

## Installation
In order to explore the Kubernetes cluster on Google Kubernetes Engine you need to install the Google Cloud SDK command line tool.


### Download the Google Cloud SDK 
  
  **Linux/Mac:** 
  
  1. Enter the following at a command prompt: `curl https://sdk.cloud.google.com | bash`
  2. Restart your shell: `exec -l $SHELL`
  3. Run gcloud init to initialize the gcloud environment: `gcloud init` 
  
  **Windows:**
  
  1. Download the [Cloud SDK installer](https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe).
  2. Launch the installer and follow the prompts.
  Cloud SDK requires Python 2 with a release version of Python 2.7.9 or later.
  3. After installation has completed, accept the following options:
        - Start Cloud SDK Shell
        - Run `gcloud init`
  The installer starts a terminal window and runs the `gcloud init` command.
  The default installation does not include the App Engine extensions required to deploy an application using gcloud commands.
    
    
### Initialize gcloud

Do **all steps** in the guide until you have typed:  `gcloud init`.
    - Authenticate in the browser when you are asked to.
    - Pick an existing project as your default for now (*option 1*):
    
    Pick cloud project to use:
     [1] arched-media-225216
     [2] Create a new project
    Please enter numeric choice or text value (must exactly match list
    item):  1

## Create a cluster
We need a cluster where we want to run our application.

You can create the cluster both in the Console view in your browser and by the gcloud command line tool.
We will use the Console to do it and also look at the equivalent gcloud command. 

  1. Visit [Google Cloud Console](https://console.cloud.google.com/) in your browser.
     Click on *Kubernetes Engine* in the left side menu. If you are asked to enable the engine, do so. Read [this](https://cloud.google.com/kubernetes-engine/kubernetes-comic/) cartoon while you wait for it to get ready.
  2. Click on the button *CREATE CLUSTER*
  3. Choose *Standard cluster*
  3. Give the cluster the name `cv-cluster`.
  4. Choose the zone `europe-north1-a` (which is in Finland).
  5. Choose Master Version: Set it to the newest available version.
  6. Next you see that you can select what machine type to use. This defines the resources each node in your cluster will have. You don't need to change this.
  7. On the right side of the create button, there are two links to get the command line and REST request. Click on these to see how you can create the same cluster without the GUI.
  8. Click Create. This will probably take several minutes. In the meantime you can move on to the next section. Notice that you can view the setup progress for your cluster in the top right corner.


# Main tasks

## Fork this repository

**If you *did create* your own Google Cloud project and cluster**; fork this repo and clone it on your laptop.
We need this to use build triggers in the next step. 

**If you *did NOT* create your own Google Cloud project**; clone this repository and jump to the assignments under [Deploy to your Kubernetes Cluster](#deploy-to-your-kubernetes-cluster).


## Docker containers
To create a deployment on Kubernetes, you need to specify at least one container for your application.
Kubernetes will on a deploy pull the image specified and create pods with this container.
Docker is the most commonly used container in Kubernetes.

In this repository you will find code for both applications in the backend and frontend directories.
Each of these folders also have their own Dockerfile.
Take a look at the docker files too see how they are built up:
  - [frontend/Dockerfile](../../frontend/Dockerfile)
  - [backend/Dockerfile](../../backend/Dockerfile)
  
Notice the `.dockerignore` files as well.
This file tells the Docker daemon which files and directories to ignore, for example the `node_modules` directory.

We could create the Docker images locally from our computer by building it with the docker deamon,
but we are going to explore build triggers in Google Cloud Platform instead.

### Build triggers
1. Go to cloud console: find **Cloud Build** in the left side menu (under tools).
If you are asked to enable the Container Build API, do so.
2. Click *Create trigger*
3. Choose Github as build source. Click *Continue*
4. Select your fork as the repository and click *Continue*
5. Now its time to specify the build trigger:
    - *Name*: Backend trigger
    - *Trigger type*: `Tag`
    -  Set tag to `cv-backend-.*`
    - Leave *Included files filter (glob)* and *Ignored files filter (glob)* empty
    - *Build Configuration*: Dockerfile
    - *Dockerfile directory*: Point to the backend Dockerfile in `backend/`
    - *Dockerfile name*: `Dockerfile`
    - *Image name*: `gcr.io/$PROJECT_ID/backend:$TAG_NAME`
   
6. Click *Create trigger*

Now, do the same thing for the frontend application.
Name it `Frontend trigger`, set tag to `cv-frontend-.*`, set the directory to be `/frontend/` and
set the Docker image to be `gcr.io/$PROJECT_ID/frontend:$TAG_NAME`.

This sets up a build trigger that listens to new commits on the master branch of your repository.
If the commit is tagged with `cv-frontend-1`, it will use the Dockerfile in the frontend directory to create a new Docker image.

7. Click on the small menu on the trigger and select *Run trigger* to test it
8. Once it is finished building, you can find the image under the *Builder Images* in the menu point.

### Test the build trigger
You tried to run the build trigger manually in the previous step.
Now you will test how it works on new commits on your GitHub repository.

#### Change the code
Open the file [backend/server.js](../../backend/server.js) and edit the JSON responses to your name, workplace and education.
You can either change the code in an editor or in GitHub directly. Commit and push your commit.

#### Publish your changes
We need to add a tag to notify our build triggers that the code has changed and need to rebuild. 
There are two ways to ad a tag:

**In the terminal**

If you commit from the git command line, the command to tag the latest commit is:

  ```
  git tag -a cv-backend-2
  git push --tags
  ```
*NB: Remember to change the latest number in your tag. If cv-backend-2 already is a tag, you should use cv-backend-3* 

**In GitHub**

You can add a tag to your directly from GitHub: 
1. In the repo, Click on *releases*, next to contributors.
2. Click on *Draft a new release*
3. Write your new tag, i.e., *cv-backend-2*
4. Create release title if you want (ex: What have you done?)
5. Click *Publish release*

**Then**

Go back to the Build triggers in Cloud Console and click on *Build history* to see whether the backend starts building.
Notice that you can follow the build log if you want to see whats going on. 

## Deploy to your Kubernetes Cluster
It's time to deploy the frontend and backend to your cluster!
The preferred way to configure Kubernetes resources is to specify them in YAML files.

In the folder [yaml/](../../yaml) you find the YAML files specifying what resources Kubernetes should create.
There are two services, one for the backend application and one for the frontend application.
Same for the deployments.

1. Open the file [yaml/backend-deployment.yaml](../../yaml/backend-deployment.yaml) and
in the field `spec.template.spec.containers.image` insert your backend Docker image full name. 
It should be something like `gcr.io/MY_PROJECT_ID/backend:TAG_NAME`, example: `gcr.io/my-kubernetes-project-1234/backend:cv-backend-1`.
 
If you did not create build triggers, use the docker image `gcr.io/arched-media-225216/backend:cv-backend-3`. 

There are a few things to notice in the deployment file:
- The number of replicas is set to 3. This is the number of pods we want running at all times
- The container spec has defined port 5000, so the Deployment will open this port on the containers
- The label `app: backend` is defined three places:
  - `metadata` is used by the service, which we will look at later
  - `spec.selector.matchLabels` is how the Deployment knows which Pods to manage
  - `spec.template.metadata` is the label added to the Pods
  
2. Open the file [yaml/frontend-deployment.yaml](../../yaml/frontend-deployment.yaml) and
in the field `spec.template.spec.containers.image` insert your frontend Docker image full name. 
It should be something like `gcr.io/MY_PROJECT_ID/frontend:TAG_NAME`. If you did not create build triggers, use `gcr.io/arched-media-225216/frontend:cv-frontend-3` instead.

2. Create the resources for the backend and frontend (from root folder in the project):
  
  ```
  kubectl apply -f ./yaml/backend-deployment.yaml
  kubectl apply -f ./yaml/frontend-deployment.yaml
  ```

3. Watch the creation of pods:
  
  ```
  watch kubectl get pods
  ```

  If you don't have `watch` installed, you can use this command instead:
  
  ```
  kubectl get pods -w
  ```

  When all pods are running, quit by `ctrl + q` (or `ctrl + c` when on Windows).

Pods are Kubernetes resources that mostly just contains one or more containers,
along with some Kubernetes network stuff and specifications on how to run the container(s).
Our pods all just contains one container. There are several use cases where you might want to specify several
containers in one pod, for example if your application is using a proxy.

The Pods were created when you applied the specification of the type `Deployment`,
which is a controller resource. 
The Deployment specification contains a desired state and the Deployment controller changes the state to achieve this.
When creating the Deployment, it will create ReplicaSet, which it owns.
The ReplicaSet will then create the desired number of pods, and recreate them if the Deployment specification changes,
e.g. if you want another number of pods running or if you update the Docker image to use.
It will do so in a rolling-update manner, which we will explore soon.

*Did you noticed that the pod names were prefixed with the deployment names and two hashes?* - The first hash is the hash of the ReplicaSet, the second is unique for the Pod.

4. Explore the Deployments:
  
  ```
  kubectl get deployments
  ```

Here you can see the age of the Deployment and how many Pods that are desired in the configuration specification,
the number of running pods, the number of pods that are updated and how many that are available.

5. Explore the ReplicaSets:
  
  ```
  kubectl get replicaset
  ```

The statuses are similar to those of the Deployments, except that the ReplicaSet have no concept of updates.
If you run an update to a Deployment, it will create a new ReplicaSet with the updated specification and
tell the old ReplicaSet to scale number of pods down to zero.

## Create services
Now that our applications are running, we would like to route traffic to them.

1. Open [yaml/backend-service.yaml](../../yaml/backend-service.yaml)
  There are a few things to notice:
    - The protocol is set to TCP, which means that the Service sends requests to Pods on this protocol. UDP is also supported
    - The spec has defined port 80, so it will listen to traffic on port 80 and sends traffic to the Pods on the same port. We could also define `targetPort` if the port on the Pods are different from the incoming traffic port
    - The label `app: backend` defines that it should route requests to our Deployment with the same label

2. Create the Services:

  ```
  kubectl apply -f ./yaml/backend-service.yaml
  kubectl apply -f ./yaml/frontend-service.yaml
  ```

2. List the created services:
  
  ```
  kubectl get service
  ```

As you can see, both services have defined internal IPs, `CLUSTER-IP`. These internal IPs are only available inside the cluster. But we want our frontend application to be available from the internet. In order to do so, we must expose an external IP.

## Exposing your app
Ok, so now what? With the previous command, we saw that we had two services, one for our frontend and one for our backend. But they both had internal IPs, no external. We want to be able to browse our application from our browser.
Lets look at another way. The Service resource can have a different type, it can be set as a LoadBalancer.

1. Open the frontend service file again
2. Set `type` to be `LoadBalancer`
3. Save and run

  ```
  kubectl apply -f ./yaml/frontend-service.yaml
  ```
  
4. Wait for an external IP:

  ```
  watch kubectl get service frontend
  ```

  or
  
  ```
  kubectl get service frontend -w
  ```

5. Visit the IP in your browser to see your amazing CV online. But something is off!
    There is no data, and if you inspect the network traffic in the browser console log, you can see that the requests to the api is responding with an error code.

    This is because the frontend application is expecting the IP of the backend Service to be set at the point of deployment.
    But since we deployed the frontend application before creating the Service objects,
    meaning there was not any IP to give the frontend container on creation time.
    
6. To fix this, we can delete the ReplicaSet for the frontend application:

   ```
   kubectl delete replicaset -l app=frontend
   ```

    By doing this, the Deployment will create a new ReplicaSet which will again create new Pods.
    At this time the backend Service exists and is given to the frontend application.

 
## Rolling updates
As you read earlier, Kubernetes can update your application without down time with a rolling-update strategy. 
You will now update the background color of the frontend application, see that the build trigger creates a new image and
update the deployment to use this in your web application.

1. Open the file [frontend/src/index.css](../../frontend/src/index.css) and edit the field `background-color` to your favourite color
2. Commit your changes
3. Create a *cv-frontend-2.0* tag like we did earlier. 
3. Go back to the cloud console in your browser and make sure that the build trigger finishes successfully
4. Update the image specification on the file [yaml/frontend-deployment.yaml](../../yaml/frontend-deployment.yaml) by adding the tag `:2.0`
5. Open a new terminal window to watch the deletion and creation of Pods:
  
  ```
  watch kubectl get pods
  ```

  If you don't have `watch` installed, you can use this command instead:

  ```
  kubectl get pods -w
  ```

  Don't close this window.

7. In the other terminal window, apply the updated Deployment specification
  
  ```
  kubectl apply -f ./yaml/frontend-deployment.yaml
  ```

and watch how the Pods are terminated and created in the other terminal window.
Notice that there are always at least one Pod running and that the last of the old Pods are first terminated when on of the new ones has the status running.

## Inspection and logging
Ok, everything looks good!
But what if you need to inspect the logs and states of your applications?
Kubernetes have a built in log feature.
Lets take a look at our backend application, and see what information we can retrieve.

1. View the logs of one container
  - First, list the pod names:
    
    ```
    kubectl get pods -l app=backend
    ```
    
    The flag `l` is used to filter by pods with the label `app=backend`.

  - Now, you can view the logs from one pod:
    
    ```
    kubectl logs <INSERT_THE_NAME_OF_A_POD>
    ```

  - You can also get all logs filtered by label.
      
    ```
    kubectl logs -l app=backend
    ```

2. Ok, the logs were fine! Lets look at the environment variables set by Kubernetes in our containers:
  
  ```
  kubectl exec -it <INSERT_THE_NAME_OF_A_POD> -- printenv
  ```

  Here you can see that we have IP addresses and ports to our frontend service.
  These IP addresses are internal, not reachable from outside the cluster.
  You can set your own environment variables in the deployment specification.
  They will show up in this list as well.

3. We can also describe our deployment, to see its statuses and pod specification:
  
  ```
  kubectl describe deployment backend
  ```
  
  Notice that `StrategyType: RollingUpdate` that we saw when we applied an updated frontend is set by default.


## DNS
A cool thing in Kubernetes is the Kubernetes DNS.
Inside the cluster, Pods and Services have their own DNS record.
For example, our backend service is reachable on `backend.<NAMESPACE>.svc.cluster.local`. If you are sending the request from the same namespace, you can also reach it on `backend`.
We will take a look at this.

1. Get your current namespace

  ```
  kubectl config view | grep namespace: 
  ```

  If there is no output, your namespace is `default`.

2. List pods to copy a pod name

  ```
  kubectl get pods -l app=frontend
  ```

2. We will run `curl` from one of our frontend containers to see that we can reach our backend internally on `http://backend.<NAMESPACE>.svc.cluster.local:5000`

  ```
  kubectl exec -it INSERT_FRONTEND_POD_NAME -- curl -v http://backend.<NAMESPACE>.svc.cluster.local:5000
  ```

  The HTTP status should be 200 along with the message "Hello, I'm alive"

3. Run `curl` from the same container to see that we can reach our backend internally on the shortname `http://backend:5000` as well

  ```
  kubectl exec -it INSERT_FRONTEND_POD_NAME -- curl -v http://backend:5000
  ```

  The output should be the same as above. 
  
4. To fix the issue where we had to delete the frontend ReplicaSet to get the internal IP for the backend Service could be avoided if we used the DNS instead. 



# Extra tasks

### Different methods to expose a service
Right now we have exposed our frontend service through an ingress. We will now look into two other ways.

#### Service type NodePort
The first way is with the service type NodePort. If we look at our frontend service, we can see that it already is defined as this type. So we are good to go then? No, not yet.

1. We will change our frontend service to be a type NodePort instead. Open the file [yaml/frontend-service.yaml](../../yaml/frontend-service.yaml)
2. Set the `type` to be `NodePort` and save
3. Apply the changes

  ```
  kubectl apply -f ./yaml/frontend-service.yaml
  ```

4. Run

  ```
  kubectl get service frontend
  ```

  We see that our service doesn't have an external IP. But what it do have is two ports, port 80 and a port in the range 30000-32767. The last port was set by the Kubernetes master when we created our service. This port we will use togheter with an external IP.

5. The nodes in our cluster all have external IPs per default. Lets use one of those.

  ```
  kubectl get nodes -o wide
  ```

6. Copy one of the external IPs from the output above along with the node port from our service:

  ```
  curl -v <EXTERNAL_IP>:<NODE_PORT>
  ```
  
  This will output `Connection failed`. This is because we haven't opened up requests on this port. Lets create a firewall rule that allows traffic on this port:
  
7. Create a firewall rule. Switch `NODE_PORT` with the node port of your service:
  
  ```
  gcloud compute firewall-rules create cv-frontend --allow tcp:NODE_PORT
  ```
  
8. Try the curl command from `6` again.  
   The output should also here be "Hello, I'm alive"

9. Do the same, but replace the IP with the external IP from one of the other nodes. It should have the same result

How does this work? The nodes all have external IPs, so we can curl them. By default, neither services or pods in the cluster are exposed to the internet, but Kubernetes will open the port of `NodePort` services on all the nodes so that those services are available on <NODE_IP>:<NODE_PORT>.

#### Create an ingress
An ingress is a Kubernetes resource that will allow traffic from outside the cluster to your services. We will now create such a resource to get an external IP and allow requests to our frontend service.

1. Open the file [yaml/ingress.yaml](../../yaml/ingress.yaml)
  Notice that we have defined that we have configured our ingress to send requests to our `frontend` service on port `8080`.
2. Create the ingress resource:
  
  ```
  kubectl apply -f ./yaml/ingress.yaml
  ```

3. Wait for an external IP to be configured

  ```
  watch kubectl get ingress cv-ingress
  ```
  
  or
  
  ```
  kubectl get ingress cv-ingress -w
  ```
  It may take a few minutes for Kubernetes Engine to allocate an external IP address and set up forwarding rules until the load balancer is ready to serve your application. In the meanwhile, you may get errors such as HTTP 404 or HTTP 500 until the load balancer configuration is propagated across the globe.

4. Visit the external IP in your peferred browser to make sure you see your awezome CV online. If you get an error, the ingress and load balacing setup might not be completed.

#### Notes on exposing your application
LoadBalancer type and the Ingress resource is dependent on your cloud provider. Google Cloud Platform supports these features, but other providers might not.


## Health checks

Kubernetes uses health checks and readiness checks to figure out the state of the pods. If you don't define any health check, Kubernetes assumes it is <INSERT>. You can define your own.
If the health check responds with an error status code, Kubernetes will asume the container is unhealthy and kill the pod. Simliary, if the readiness check is unsuccessful, Kubernetes will asume it is not ready, and wait for it.


### Endpoint

The first way to define a health check is to define which endpoint the check should use. Our backend application contains the endpoint `/healthz`. Go ahead and define this as the health-endpoint in the backend deployment file, under the container spec:

```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: X-Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
```




# Clean up

The cluster you have created will charge your credit card after some time if you keep it running. You can use the [Google Cloud Price Calculator](https://cloud.google.com/products/calculator/) to find out how much it will cost you. If you keep it running, it will cost you money then the price is more than $300 (which is the included credits in the Free Tier).

**Close your billing account**

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Select **Billing** in the main menu
3. Select **My billing accounts**
4. Click on the menu on your billing account and click *close*

**Delete your project** 

```
gcloud config get-value project
gcloud projects delete $(!!)
```

And your are done and your credit card will not be charged.

And that's it! ⎈

## Any questions?

Contact us on [@linemoseng](https://twitter.com/linemoseng) or [@ingridguren](https://twitter.com/ingridguren).
