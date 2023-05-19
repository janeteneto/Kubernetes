# Kubernetes (K8)

![image](https://github.com/janeteneto/Kubernetes/assets/129942042/f4ac0de2-cb27-4be5-b776-8e758d0a9837)

- Kubernetes (K8s) is a powerful container orchestration platform that provides extensive features for managing and scaling containerised applications.

## How does K8 works

![image](https://github.com/janeteneto/Kubernetes/assets/129942042/6ccac873-6f07-43bb-b9a7-f63990840a3b)

- Kubernetes (K8s) works by managing containers, allowing you to run and scale applications across a cluster of machines. 
- At its core, K8s uses a master-worker architecture. The **master node** controls the cluster and manages the overall state, while the **worker nodes** run the containers. You define the desired state of your application in a **YAML or JSON file, known as a manifest**, which includes details like the number of **replicas**, resource requirements, and networking rules. 
- The master node continuously monitors the cluster, ensuring that the current state matches the desired state specified in the manifest. It schedules containers onto available worker nodes, allocates resources, handles networking, and automatically restarts containers if they fail. 
- K8s also provides additional features such as load balancing, scaling, and rolling updates, making it easier to manage and scale applications in a dynamic environment.

## Benefits of K8

- Scalability - automatically adjust the number of running containers, ensuring your application can handle increased traffic or workload without manual intervention.
- High Availability - automatically restart containers that fail, distribute containers across multiple nodes to avoid single points of failure, and handle load balancing to evenly distribute traffic.
- Resource Efficiency - 
- Self-Healing -
- Service Discovery and Load Balancing -

## Who is using K8

- Google, Microsoft, AWS, Spotify, Airbnb, etc.


## When not to use Microservices

- It may not be suitable when the application is small and relatively simple, with limited functionality and low scalability requirements.
- Adopting a microservices architecture can introduce unnecessary complexity and overhead, as the benefits of independent scalability and flexibility might not outweigh the additional development and operational complexities associated with managing multiple services.

## When not to use Kubernetes 

- When you have a small, simple application with few components and a low demand for scalability. Kubernetes introduces additional complexity and overhead, which may not be justified for such straightforward applications. In these cases, simpler deployment options like deploying directly to a virtual machine or using a lighter-weight container orchestration solution may be more appropriate.


## K8 Setup

1. Open Docker --> Click Settings --> Kubernetes --> tick `enable Kubernetes` and `Show system containers` --> press Apply & restart

2. Once Kubernetes fully starts, open a terminal and run `kubectl get svc`

### Create a deployment for nginx

1. We start bye creating a new yml file called `nginx-deployment.yml`. This can be located anywhere on your local user.

2. Add the following script to it:
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
       app: nginx
    spec:
      containers:
      - name: nginx
        image: janeteprofile/nginx:v1
        ports:
        - containerPort: 80
  ````
  
  - Make sure to indent properly. Whenever you want to indent, just double click the space bar.
  - This file gives the conf for the deployment using kubernetes

3. On your terminal, on the same location as the nginx-deployment.yml, run `kubectl create -f nginx-deploy.yml`

4. Then run `kubectl get deploy` to see the deployments and their state

5. Run ` kubectl get pods` to see the pods created (they work like instances)

### Create a deployment for node

1. On the same directory, create a file named `node-deploy.yml` with the following script:

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-deployment
spec:
  selector:
    matchLabels:
      app: node
  replicas: 3
  template:
    metadata:
      labels:
       app: node
    spec:
      containers:
      - name: node
        image: janeteneto/app:v1
        ports:
        - containerPort: 80
        env:
          - name: DB_HOST
            value: mongodb://10.107.199.89:27017/posts
        
---

apiVersion: v1
kind: Service
metadata: 
  name: node-svc
  namespace: default
spec:
  ports:
  - nodePort: 30002
    port: 3000
    targetPort: 3000
  selector:
    app: node
  type: LoadBalancer
  ````
  
- This script includes the deployment **and** the service for node.
- The image should be the image of the container with the app
-  The ip on **value** is Mongodb's ip. You can check the ip by running `kubectl get svc`
  
2. On the terminal where your file is, run `kubectl create -f node-deploy.yml` to create both the deployment and service

3. Go to localhost:30002 and the app should be running

### Create a deployment for mongoDB
  
1. Create a new file called `mongodb-pvc.yml` and add the following script:
  ````
  apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
````

2. Then let's create the deployment and service in a file called `mongo-deploy.yml` with the following code:
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
spec:
  selector:
    matchLabels:
      app: mongodb
  replicas: 3
  template:
    metadata:
      labels:
       app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongodb
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      volumes:
      - name: mongodb-data
        persistentVolumeClaim:
          claimName: mongodb-pvc
---

apiVersion: v1
kind: Service
metadata: 
  name: mongodb-svc
  namespace: default
spec:
  ports:
  - nodePort: 30003
    port: 27017
    targetPort: 27017
  selector:
    app: mongodb
  type: NodePort
  ````
  
- This also attaches the `pvc`

3. Run `kubectl create -f mongo-deploy.yml`

4. Run `kubectl exec node_port_number -- env node seeds/seed.js`

- After this, you should see a message on the terminal saying: 
````
Database Cleared
Database Seeded
````

5. Go to localhosts:30002/posts and posts should be running!
