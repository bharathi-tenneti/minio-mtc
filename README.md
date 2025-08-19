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
NAME             HOST/PORT                                                    PATH   SERVICES        PORT      TERMINATION     WILDCARD
minio-route      minio-route-minio.apps.cluster1.sandbox1882.opentlc.com             minio-service   console   edge/Redirect   None
minio-s3-route   minio-s3-route-minio.apps.cluster1.sandbox1882.opentlc.com          minio-service   api       edge/Redirect   None

```
Use the "minio-route" to access MinIO web UI, and create a bucket with any arbitrary name. You can use "minioadmin" for creds temporarily.

(images/minio.png)


# Usecase 2: Create bucket using ODF Nooba Storage class
If you have installed ODF in your target cluster, you can directly use the S3 endpoint exposed by the noobaa storage class.

Create ObjectBucketClaim in the existing ODF on target cluster.
```
oc create -f obc.yaml
```


# Configure MTC UI -  Create clusters 

```
oc get routes -n openshift-migration
```
This route will give you MTC UI, configure source cluster, and target cluster (if different from host).

```
oc patch configs.imageregistry.operator.openshift.io/cluster \
  --patch '{"spec":{"defaultRoute":true}}' --type=merge

oc get routes -n openshift-image-registry

```
For "Direct Image" migration to work, both clusters should expose the internal registries via route.
For "host" cluster, you can explicitly provide this in the host cluster configuration 
```
oc get migcluster -n openshift-migration
oc edit migcluster host -n openshift-migration

```
```
spec:
  isHostCluster: true
  exposedRegistryPath: <your-host-registry-route-from-above>
```

[!images](images/mtc-clusters.png)

# Configure MTC UI - Create Replication repository.
Option 1: Pick "S3" option here, if using Minio for bucket , you can use the "minio-s3-route" as the S3 Endpoint, along with "minioadmin" creds as the AWS creds, along with the bucket name you have created using minio UI.

[!images](images/s3-minio-mtc.png)

Option 2: Pick "S3" for ODF bucket as well.
_You can look at the secret and configmap that contains the S3 AWS Access Key, and AWS Secret Access, and S3 Endpoint details. Note : Use S3 route as it should be accessible from the source cluster as well.
_
```
$ oc get secrets -n openshift-migration
$ oc get cm -n openshift-migration

$ oc get routes -n openshift-storage 
s3            s3-openshift-storage.apps.cluster-j8s6v.j8s6v.sandbox230.opentlc.com                   

```

[!images](images/s3-odf-mtc.png)


# Create migration plan 
Create migration plans by choosing which projects to move across to target cluster. Keep in mind to have compatible storage classes for  PVCs across clusters. 
You can "stage" a migration, and the "cutover" applications from source cluster.
_Note: If your moving your apps managed by ArgoCD, do not "Halt" these apps during migration from source cluster, as ArgoCD will constantly try to sync them up._

[!image](images/mtc-plans.png)
