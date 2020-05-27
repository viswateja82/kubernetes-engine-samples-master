# Using Persistent Disks with WordPress and MySQL

# POC on Wordpress running on GCP

 I am gonna show how to set up a single-replica WordPress deployment on Google Kubernetes Engine (GKE) using a MySQL database. Instead of installing MySQL, I used Cloud SQL, which provides a managed version of MySQL. WordPress uses PersistentVolumes (PV) and PersistentVolumeClaims (PVC) to store data.

https://github.com/viswateja82/kubernetes-engine-samples-master/blob/master/screenshots/architecutre%20diagram.jpeg

 <b> Objectives </b>

 Create a GKE cluster.
 Create a PV and a PVC backed by Persistent Disk.
 Create a Cloud SQL for MySQL instance.
 Deploy WordPress.
 Set up your WordPress blog.

Initially,I enabled the GKE and Cloud SQL Admin APIs-

<i>-> gcloud services enable container.googleapis.com sqladmin.googleapis.com </i>

Downloaded the manifest files from the GitHub repository -

<i>-> git clone https://github.com/viswateja82/kubernetes-engine-samples-master </i>

Now, lets create a GKE cluster to host your WordPress app container, named persistent-disk-tutorial that has three nodes-

<i>-> CLUSTER_NAME=persistent-disk-tutorial
-> gcloud container clusters create $CLUSTER_NAME --num-nodes=3 --enable-autoupgrade --no-enable-basic-auth --no-issue-client-certificate --enable-ip-alias --metadata disable-legacy-endpoints=true </i>

<i>-> gcloud container clusters get-credentials persistent-disk-tutorial </i>

<b> Creating a PV and a PVC backed by Persistent Disk </b>

To create the storage required for WordPress, we need to create a PVC. GKE has a default StorageClass resource installed that lets you dynamically provision PVs backed by Persistent Disk. 

I deployed the wordpress-volumeclaim.yaml file to create the PVCs required for the deployment.

This manifest file describes a PVC that requests 20 GB of storage. A StorageClass resource hasn't been defined in the file, so this PVC uses the default StorageClass resource to provision a PV backed by Persistent Disk.

<i>-> kubectl apply -f $WORKING_DIR/wordpress-volumeclaim.yaml </i>

<b> Creating a Cloud SQL for MySQL instance </b>

Now, lets create an instance named mysql-wordpress-instance

<i>-> INSTANCE_NAME=mysql-wordpress-instance
-> gcloud sql instances create $INSTANCE_NAME </i>

Now, lets make sure that both GKE cluster and MySQL instance is in same region.

Add the instance connection name as an environment variable -

<i>-> export INSTANCE_CONNECTION_NAME=$(gcloud sql instances describe $INSTANCE_NAME --format='value(connectionName)') </i>

Create a database user called wordpress and a password for WordPress to authenticate to the instance -

<i>-> CLOUD_SQL_PASSWORD=$(openssl rand -base64 18)
-> gcloud sql users create wordpress --host=% --instance $INSTANCE_NAME --password $CLOUD_SQL_PASSWORD </i>

<b> Deploying WordPress </b>

Before we deploy WordPress, we need to create a service account. So, I created a Kubernetes secret to hold the service account credentials and another secret to hold the database credentials.

To allow WordPress app to access the MySQL instance through a Cloud SQL proxy, I created a service account -

<i>-> SA_NAME=cloudsql-proxy
-> gcloud iam service-accounts create $SA_NAME --display-name $SA_NAME </i>

Added the service account email address as an environment variable -

<i>-> SA_EMAIL=$(gcloud iam service-accounts list --filter=displayName:$SA_NAME --format='value(email)') </i>

Added the cloudsql.client role to your service account -

<i>-> gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --role roles/cloudsql.client --member serviceAccount:$SA_EMAIL </i>

Created a key for the service account -

<i>-> gcloud iam service-accounts keys create $WORKING_DIR/key.json --iam-account $SA_EMAIL </i>

Now, we have downloaded copy of the key.json file in current directory.

Created a Kubernetes secret for the MySQL credentials -

<i>-> kubectl create secret generic cloudsql-db-credentials --from-literal username=wordpress --from-literal password=$CLOUD_SQL_PASSWORD </i>

Created a Kubernetes secret for the service account credentials -

<i>-> kubectl create secret generic cloudsql-instance-credentials --from-file $WORKING_DIR/key.json </i>

Now, its time to do some magic, deploy  WordPress container in the GKE cluster -

The wordpress_cloudsql.yaml manifest file describes a deployment that creates a single pod running a container with a WordPress instance. This container reads the WORDPRESS_DB_PASSWORD environment variable that contains the cloudsql-db-credentials secret that we created in previous step.

This manifest file also configures the WordPress container to communicate with MySQL through the Cloud SQL proxy running in the sidecar container. The host address value is set on the WORDPRESS_DB_HOST environment variable.

Prepare the deployment file by replacing environment variables -

<i>-> cat $WORKING_DIR/wordpress_cloudsql.yaml.template | envsubst > $WORKING_DIR/wordpress_cloudsql.yaml </i>

Deployed the wordpress_cloudsql.yaml manifest file -

<i>-> kubectl create -f $WORKING_DIR/wordpress_cloudsql.yaml </i>

Now, you can see the pod is running,

<i>-> kubectl get pod -l app=wordpress --watch </i>

<b> Expose the WordPress service </b>

I deployed a WordPress container, but it's currently not accessible from outside my cluster because it doesn't have an external IP address. I am going to expose my WordPress app to traffic from the internet by creating and configuring a load balancer.

Created a service of type:LoadBalancer - 

<i>-> kubectl create -f $WORKING_DIR/wordpress-service.yaml </i>

Now the service will have external Ip assigned -

<i>-> kubectl get svc -l app=wordpress --watch 

viswa_teja@cloudshell:~ (sep-poc-internal)$ kubectl get svc
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      10.28.0.1     <none>         443/TCP        13h
wordpress    LoadBalancer   10.28.0.168   35.223.86.39   80:32601/TCP   12h
</i>

Now I have a blog site. Go to the following URL:

<i>-> http://35.223.86.39 </i>

Please check screenshots folder.
