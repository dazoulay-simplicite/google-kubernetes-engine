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

Docker images registry
----------------------

Configure the registry:

	gcloud auth configure-docker

Tag the chosen Simplicité private image (previously pulled from DockerHub):

	docker tag simplicite/platform:<tag, e.g. 5-beta> <server, e.g. gcr.io>/<project ID>/simplicite/platform:<tag, e.g. 5-beta>

And push it to the regitry:

	docker push <server, e.g. gcr.io>/<project ID>/simplicite/platform:<tag, e.g. 5-beta>

PostgreSQL database
-------------------

### Persistent disk

Create a persistent disk for the database data with:

	gcloud compute disks create postgresql-disk --size 50GB --zone <zone e.g. us-central1-c>

### Persistent Volume

Create the volume with:

	kubectl apply -f ./postgresql/volume.yml

Check the persistent volume status with `kubectl get pv`.

### Persistent volume claim

Create the volume claim with:

	kubectl apply -f ./postgresql/volume-claim.yml

Check the persistent volume claim status with `kubectl get pvc`.

### Deployment

Create the deployment with:

	kubectl apply -f ./postgresql/deployment.yml

Check the deployment status with `kubectl get pods`.

### Service

Create the service with:

	kubectl apply -f ./postgresql/service.yml

Check the deployment status with `kubectl get services`.

MySQL database
--------------

**TODO**

Simplicité platform
-------------------

### Persistent disk for the Git repositories

Create a persistent disk for the Git repositories of your Simplicité instance with:

	gcloud compute disks create simplicite-git-disk --size 10GB --zone <zone e.g. us-central1-c>

### Persistent volume for the Git repositories

Create the volume with:

	kubectl apply -f ./simplicite/git-volume.yml

Check the persistent volume status with `kubectl get pv`.

### Persistent volume claim for the Git repositories

Create the volume claim with:

	kubectl apply -f ./simplicite/git-volume-claim.yml

Check the persistent volume claim status with `kubectl get pvc`.

### Deployment

Create the deployment with:

	IMAGE=<image tag> DB=<mysql|postgresql> envsubst < ./simplicite/deployment.yml | kubectl apply -f -

where the image tag matches tag of the image you have pushed to the registry (see above).

Check the deployment status with `kubectl get pods`.

### Service

Create the service with:

	kubectl apply -f ./simplicite/service.yml

Check the deployment status with `kubectl get services`.
