# Wanderlust Deployment on Kubernetes

## Steps for Kubernetes deployment:
1) Create kubernetes namespace :
    ```bash
    kubectl create namespace wanderlust
    ```

2) Update kubernetes config context : 
    ```bash
    kubectl config set-context --current --namespace wanderlust
    ```

3) Enable DNS resolution on kubernetes cluster :

- Check coredns pod in kube-system namespace and you will find <i> Both coredns pods are running on master node </i>

    ```bash
    kubectl get pods -n kube-system -o wide | grep -i core
    ```
- Above step will run coredns pod on worker node as well for DNS resolution

    ```bash
    kubectl edit deploy coredns -n kube-system -o yaml
    ```
<i> Make replica count from 2 to 4 </i>

4) Navigate to frontend directory :
    ```bash
    cd frontend
    ```

5) Edit .env.docker file and change the public IP Address with your worker node public IP :
    ```bash
    vi .env.docker
    ```

6) Build frontend docker image : 
    ```bash
    docker build -t user_name/frontend_image_name:tag .
    ```

7) Navigate to backend directory :
    ```bash
    cd ../backend/
    ```

8) Open .env.docker file and edit below variables : 

    - MONGODB_URI: \<your-mongodb-servicename>
    - REDIS_URL: \<your-redis-servicename>
    - FRONTEND_URL: \<your-workernode-publicIP>

> Note: To get service names, check <u>mongodb.yaml, redis.yaml</u>

9) Build backend docker image : 
    ```bash
    docker build -t user_name/backend_image_name:tag .
    ```

#
10) Once, Image is pushed to DockerHub, navigate to kubernetes directory and Apply manifests file in below order:

    ```bash
    cd ../kubernetes
    kubectl apply -f persistentVolume.yaml 
    kubectl apply -f persistentVolumeClaim.yaml 
    kubectl apply -f mongodb.yaml 
    ```
    > Note: Now Wait for 3-4 mins to get mongodb, redis pods and service should be up, otherwise backend-service will not connect.
    ```bash
    kubectl apply -f redis.yaml 
    kubectl apply -f backend.yaml 
    kubectl apply -f frontend.yaml
    ```
11) Check all deployments and services :
    ```bash 
    kubectl get all
    ```

12) Check logs for all the pods :

    > Note: This is mandatory to ensure all pods and services are connected or not, if not then recreate deployments
    ```bash
    kubectl logs <pod-name>
    ```

13) Navigate to chrome and access your application at 31000 port :
    ```bash
    http://<your-workernode-publicip>:31000/
    ```
