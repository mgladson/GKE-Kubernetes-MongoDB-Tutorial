# GCP/GKE Kubernetes-MongoDB-Tutorial

[0] Requirements

(0) Google SDK
```
gcloud auth login
gcloud config set project <project_id>
```

(1) kubectl
```
gcloud components install kubectl
```

Create a K8S cluster 
```
gcloud beta container --project "<project_id>" clusters create "cluster-1" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.22.8-gke.202" --release-channel "regular" --machine-type "e2-custom-4-8192" --image-type "COS_CONTAINERD" --disk-type "pd-ssd" --disk-size "10" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "1" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-private-nodes --master-ipv4-cidr "172.16.0.0/28" --enable-ip-alias --network "projects/kubernetes-bf-chainlink/global/networks/default" --subnetwork "projects/kubernetes-bf-chainlink/regions/us-central1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "us-central1-c"
```
Once the K8S cluster is created, google sdk will return the following
```
NAME       LOCATION       MASTER_VERSION  MASTER_IP     MACHINE_TYPE      NODE_VERSION    NUM_NODES  STATUS
cluster-1  us-central1-c  1.22.8-gke.202  35.225.229.3  e2-custom-4-8192  1.22.8-gke.202  1          RUNNING
```

To login to the cluster
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project <project_id>

To confirm that you are successfully connected, run the following commands to check the nodes in your cluster, and then view the pods running in the kube-system namespace
```
kubectl get nodes
kubectl get pods -n kube-system
```

Create mongo-secret.yaml file to store the mongoDB username+password

```
kubectl apply -f "mongo-secret.yaml"
```
Confirm the secret
```
kubectl get secret
```
If the secret was correctly applied, google SDK will output:
```
NAME                  TYPE                                  DATA   AGE
default-token-9chjn   kubernetes.io/service-account-token   3      27m
mongodb-secret        Opaque                                2      45s
```

Create mongo.yaml file for the deployment and service.

Get `containerPort` from https://hub.docker.com/_/mongo

```
kubectl apply -f "mongo.yaml"
```
Confirm the service is running
```
kubectl get service
```
If the service was correctly started, google SDK will output:
```
NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.8.0.1     <none>        443/TCP     111m
mongodb-service   ClusterIP   10.8.9.28    <none>        27017/TCP   26s
```

Check the IP endpoint matches with the two following commands:
```
kubectl describe service mongodb-service
```
```
Name:              mongodb-service
Namespace:         default
Labels:            <none>
Annotations:       cloud.google.com/neg: {"ingress":true}
Selector:          app=mongodb
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.8.9.28
IPs:               10.8.9.28
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         10.4.0.9:27017
Session Affinity:  None
Events:            <none>
```

Verify pod IP: 10.4.0.9
```
kubectl get pod -o wide
```
```
NAME                                 READY   STATUS    RESTARTS   AGE   IP         NODE                                       NOMINATED NODE   READINESS GATES
mongodb-deployment-8f6675bc5-7nj2v   1/1     Running   0          67m   10.4.0.9   gke-cluster-1-default-pool-419e8f36-1dp5   <none>           <none>
```

Create mongo.yaml file for the deployment and service

Get `.env` and `containerPort` from https://hub.docker.com/_/mongo-express

```
kubectl apply -f "mongo-express.yaml"
```
Verify pod creation:
```
kubectl get pod
```
```
NAME                                 READY   STATUS    RESTARTS   AGE
mongo-express-78fcf796b8-zwwvw       1/1     Running   0          18s
mongodb-deployment-8f6675bc5-7nj2v   1/1     Running   0          76m
```
Verify mongo-express logs:
```
kubectl logs mongo-express-78fcf796b8-zwwvw
```
```
(node:7) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
Mongo Express server listening at http://0.0.0.0:8081
←[31mServer is open to allow connections from anyone (0.0.0.0)←[39m
←[31mbasicAuth credentials are "admin:pass", it is recommended you change this in your config.js!←[39m
```
Interact with mongo DB at the following url: http://35.202.178.243:8081/
