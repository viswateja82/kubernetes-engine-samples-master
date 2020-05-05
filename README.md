# Using Persistent Disks with WordPress and MySQL

# POC on Wordpress running on GCP

 I am gonna show how to set up a single-replica WordPress deployment on Google Kubernetes Engine (GKE) using a MySQL database. Instead of installing MySQL, I used Cloud SQL, which provides a managed version of MySQL. WordPress uses PersistentVolumes (PV) and PersistentVolumeClaims (PVC) to store data.

 <b> Objectives </b>

 Create a GKE cluster.
 Create a PV and a PVC backed by Persistent Disk.
 Create a Cloud SQL for MySQL instance.
 Deploy WordPress.
 Set up your WordPress blog.

Initially,I enabled the GKE and Cloud SQL Admin APIs-

<i> gcloud services enable container.googleapis.com sqladmin.googleapis.com </i>

Downloaded the manifest files from the GitHub repository-

<i> git clone https://github.com/viswateja82/kubernetes-engine-samples-master </i>

Now, elts create a GKE cluster to host your WordPress app container, named persistent-disk-tutorial that has three nodes-

<i> CLUSTER_NAME=persistent-disk-tutorial
gcloud container clusters create $CLUSTER_NAME --num-nodes=3 --enable-autoupgrade --no-enable-basic-auth --no-issue-client-certificate --enable-ip-alias --metadata disable-legacy-endpoints=true </i>

<i>  gcloud container clusters get-credentials persistent-disk-tutorial </i>

<b> Creating a PV and a PVC backed by Persistent Disk </b>

To create the storage required for WordPress, we need to create a PVC. GKE has a default StorageClass resource installed that lets you dynamically provision PVs backed by Persistent Disk. 

I deployed the wordpress-volumeclaim.yaml file to create the PVCs required for the deployment.

This manifest file describes a PVC that requests 20 GB of storage. A StorageClass resource hasn't been defined in the file, so this PVC uses the default StorageClass resource to provision a PV backed by Persistent Disk.

<i> kubectl apply -f $WORKING_DIR/wordpress-volumeclaim.yaml </i>

<b> Creating a Cloud SQL for MySQL instance </b>

Now, lets create an instance named mysql-wordpress-instance

<i>  INSTANCE_NAME=mysql-wordpress-instance
gcloud sql instances create $INSTANCE_NAME </i>

Now, lets make sure that both GKE cluster and MySQL instance is in same region.

Add the instance connection name as an environment variable -

<i> export INSTANCE_CONNECTION_NAME=$(gcloud sql instances describe $INSTANCE_NAME --format='value(connectionName)') </i>

Create a database user called wordpress and a password for WordPress to authenticate to the instance -

<i> CLOUD_SQL_PASSWORD=$(openssl rand -base64 18)
gcloud sql users create wordpress --host=% --instance $INSTANCE_NAME --password $CLOUD_SQL_PASSWORD </i>

