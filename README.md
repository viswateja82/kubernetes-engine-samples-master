# Using Persistent Disks with WordPress and MySQL

# wordpress_gcp
 POC on Wordpress running on GCP

 I am gonna show how to set up a single-replica WordPress deployment on Google Kubernetes Engine (GKE) using a MySQL database. Instead of installing MySQL, I used Cloud SQL, which provides a managed version of MySQL. WordPress uses PersistentVolumes (PV) and PersistentVolumeClaims (PVC) to store data.

 # Objectives

 Create a GKE cluster.
 Create a PV and a PVC backed by Persistent Disk.
 Create a Cloud SQL for MySQL instance.
 Deploy WordPress.
 Set up your WordPress blog.

