![Simplicit&eacute; Software](https://www.simplicite.io/resources/logos/logo250.png)
* * *

Simplicite + PostgreSQL on Google Kubernetes Engine (GKE)
=========================================================

Prerequisites
-------------

Install and configure the [Google cloud SDK](https://cloud.google.com/sdk/docs/install).

And install the `kubectl` Kubernetes CLI:

	gcloud components install kubectl

Project
-------

Create a new project (or use an existing one):

	gcloud config set project <project ID>

Cluster
-------

Create a node cluster with at least 2 nodes with:

	gcloud container clusters create <cluster name, e.g. test-cluster> --num-nodes=<number of nodes, e.g. 2> --machine-type <type, e.g. g1-small> --zone <zone, e.g. us-central1-c>

Check the nodes status with `kubectl get nodes`.

And connect to the cluster with:

	gcloud container clusters get-credentials <cluster name, e.g. test-cluster> --zone <zone e.g us-central1-c>

Docker registry
---------------

	gcloud auth configure-docker

Tag the chosen Simplicité private image (previously pulled from DockerHub):

	docker tag simplicite/platform:<tag, e.g. 5-beta> <server, e.g. gcr.io>/<project ID>/simplicite/platform:<tag, e.g. 5-beta>

And push it to the regitry:

	docker push <server, e.g. gcr.io>/<project ID>/simplicite/platform:<tag, e.g. 5-beta>

PostgreSQL
----------

### Persistent disk

Create a persistent disk for the database data with:

	gcloud compute disks create postgresql-disk --size 50GB --zone <zone e.g. us-central1-c>


### Persistent Volume

Create the volume with:

	kubectl apply -f ./postgresql/postgres-volume.yml

Check the persistent volume status with `kubectl get pv`.

### Persistent volume claim

Create the volume claim with:

	kubectl apply -f ./postgresql/postgres-volume-claim.yml

Check the persistent volume claim status with `kubectl get pvc`.

### Deployment

Create the deployment with:

	kubectl apply -f ./postgresql/postgres-deployment.yml

Check the deployment status with `kubectl get pods`.

### Service

Create the service with:

	kubectl apply -f ./postgresql/postgres-service.yml

Check the deployment status with `kubectl get services`.

Simplicite
----------

### Persistent disk

Create a persistent disk for the Git repositories of your Simplicité instance with:

```bash
gcloud compute disks create <disk name, e.g. simplicite-git-disk> \
 --size 10GB \
 --zone <zone e.g. us-central1-c>
```

### Persistent volume

Create a `simplicite-git-volume.yml`file with the following content:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: simplicite-git-volume
  labels:
    name: simplicite-git-volume
spec:
  capacity:
    storage: 10Gi
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: simplicite-git-disk
    fsType: ext4
```

And create it with:

```bash
kubectl apply -f simplicite-git-volume.yml
```

Check the persistent volume status with `kubectl get pv`.

### Persistent volume claim

Create a `simplicite-git-volume-claim.yml`file with the following content:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: simplicite-git-volume-claim
  labels:
    type: local
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: simplicite-git-volume
```

And create it with:

```bash
kubectl apply -f simplicite-git-volume-claim.yml
```

Check the persistent volume claim status with `kubectl get pvc`.

### Push image to the registry

### Deployment

Create a `simplicite-deployment.yml`file with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simplicite
  labels:
    app: simplicite
spec:
  replicas: 1
  selector:
    matchLabels:
      app: simplicite
  template:
    metadata:
      labels:
        app: simplicite
    spec:
      containers:
        - name: simplicite
          image: <server, e.g. gcr.io>/<project ID>/simplicite/server:<tag, e.g. 5-beta>
          volumeMounts:
          - name: simplicite-git-volume-mount
            mountPath: /usr/local/tomcat/webapps/ROOT/WEB-INF/git
            subPath: simplicite-git
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: postgres
            - name: DB_VENDOR
              value: postgresql
            - name: DB_NAME
              value: simplicite
            - name: DB_USER
              valuesimplicite
            - name: DB_PASSWORD
              value: simplicite
            - name: DB_SETUP
              value: "true"
            - name: DB_WAIT
              value: "100"
      restartPolicy: Always
      volumes:
      - name: simplicite-git-volume-mount
        persistentVolumeClaim:
          claimName: simplicite-git-volume-claim
```

And create it with:

```bash
kubectl apply -f simplicite-deployment.yml
```

Check the deployment status with `kubectl get pods`.

### Service

Create a `simplicite-service.yml`file with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: simplicite
  name: simplicite
spec:
  type: LoadBalancer
  sessionAffinity: ClientIP
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: simplicite
status:
  loadBalancer: {}
```

And create it with:

```bash
kubectl apply -f simplicite-service.yml
```

Check the deployment status with `kubectl get services`.

