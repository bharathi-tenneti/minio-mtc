# minio-mtc/odf
How to migrate sample application between Openshift clusters, using Migration toolkit for Containers (MTC), and configuring Replication repository using Minio/ODF usecases.

# Pre-Reqs
1. Running Openshift clusters, source and target.
2. Have atleast one application workload to demo migration.
3. For ODF usecase, install ODF on both clusters.
4. Install MTC on both clusters, same version.
5. MinIO Usecase needs Minio service running in target cluster.

# Usecase 1: Create bucket using MinIO 
Create MinIO Deployment on the target cluster, and also use MTC GUI console from the target cluster to kick off migration plan.
This should create UI, and API routes for MinIO. Note that in this example, we use default "minioadmin" as user and password for the bucket creation, feel free to change as needed.

```
oc create -f minio-deploy.yaml
oc get routes -n minio
```
[!image](minio.png)
[!image](minio-routes.png)

# Usecase 2: Create bucket using ODF Nooba Storage class
If you have installed ODF in your target cluster, you can directly use the S3 endpoint exposed by the noobaa storage class.

Create ObjectBucketClaim in the existing ODF on target cluster.
```
oc create obc.yaml
```
You can look at the secret and configmap that contains the S3 AWS Access Key, and AWS Secret Access IS, and S3 Endpoint details.
```
oc get secrets -n openshift-storage
os get cm -n openshift-storage
```


