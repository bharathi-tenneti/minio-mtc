# minio-mtc
How to migrate sample application between Openshift clusters, using Migration toolkit for Containers (MTC), and  MinIO as Replication repository .

# Pre-Reqs
1. Running Openshift clusters, source and target.
2. Have atleast one application workload to demo migration.
3. For this example usecase, install ODF on both clusters.
4. Install MTC on both clusters, same version.

## Create MinIO Deployment on the target cluster, and also use MTC GUI console from the target cluster ti kick off migration plan.
```
oc create -f minio-deploy.yaml
```
This should create UI, and APi routes for MinIO.
Note tha in this example, we use default "miniadmin" as user and password for the bucket creation, feel free to change as needed.

