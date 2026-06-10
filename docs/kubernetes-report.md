# CampusFit Kubernetes Deployment Report



## 1. K8s vs Docker Networking

In Docker Compose, containers communicate using service names defined in docker-compose.yml. Kubernetes works similarly but uses Service objects. Each Service gets a DNS name inside the cluster — for example, our MongoDB Service named "mongodb-service" is reachable at mongodb-service:27017 from any pod. Unlike Docker, Kubernetes separates networking (Service) from compute (Deployment) as distinct objects. ClusterIP services are internal only, while LoadBalancer exposes the
app externally. Docker Desktop automatically maps LoadBalancer to localhost, making testing easy.



## 2. How PVC Works

A PersistentVolumeClaim (PVC) is a request for storage. When we created mongodb-pvc.yaml requesting 1Gi, Kubernetes automatically provisioned a PersistentVolume using Docker Desktop's hostpath storage class. MongoDB mounts this volume at /data/db. When the MongoDB pod was deleted, the new pod mounted the same PVC — so all student data (Alice, Bob, PersistenceTest) survived. Without PVC, data would be lost every time the pod restarts because container filesystems are ephemeral.



## 3. Scaling Behavior

Scaling from 2 to 3 replicas was instant with kubectl scale. Kubernetes created a new pod and the Service automatically load balanced traffic across all 3. When APP\_ENV was updated and a rolling restart triggered, Kubernetes replaced pods one by one — ensuring zero downtime. Old pods were only terminated after new ones became Ready (confirmed by health probes on /health). Rolling back was equally seamless using kubectl rollout undo.



## 4. Challenges Faced

The main challenge was setting up k3s inside a Docker container — it failed due to missing kernel capabilities. The solution was using Docker Desktop's built-in Kubernetes instead, which worked perfectly. Another learning was understanding that ConfigMap changes don't automatically restart pods — a manual kubectl rollout restart was needed to pick up the APP\_ENV change. The intentional break (wrong image) showed ImagePullBackOff clearly, and fixing it with kubectl set image was straightforward.

