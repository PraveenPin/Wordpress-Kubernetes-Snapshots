# Prerequisites:
1. EKS Cluster
2. Minimum Kubernetes Version is 1.21

# ToDo
1. Install the EBS CSI Driver AddOn
2. Install the External Snapshotter
3. Configure the Amazon EBS CSI plugin to use IAM roles for service accounts
4. Creating manifest files for snapshots

## Install the CSI Driver
1. Container Storage Interface
2. To decouple the lifecycle of storage implementations from the Kubernetes project itself, the Container Storage Interface (CSI) was created. This is a standard for implementing storage-specific solutions for container-based applications, exposing various block and file storage systems to workloads on container orchestration systems, such as Kubernetes.
3. This CSI driver enables container orchestrators (such as Kubernetes) to manage the lifecycle of Amazon EBS volumes. When used with Amazon EKS, this driver is installed on a cluster and storage resources can be specified.
4.Amazon EKS add-ons provide installation and management of a curated set of add-ons for Amazon EKS clusters.
5. Can be installed through:
    1. Amazon EKS Console
    2. AWS CLI
    3. Amazon EKS API (eksctl, Terraform)
6. Add-on Required:
    1. aws-ebs-csi-driver

## Install the External Snapshotter
1. To use the snapshot functionality of the Amazon EBS CSI driver, you must install the external snapshotter before installing the add-on. The external snapshotter is provided by the Kubernetes CSI Special Interest Group (SIG) and has components that must be installed in the following order:
    1. CustomResourceDefinition (CRD) for volumesnapshotclasses, volumesnapshots, and volumesnapshotcontent
    2. RBAC (ClusterRole, ClusterRoleBinding, and so on)
    3. controller deployment
2. The EBS CSI driver includes the csi-snapshotter sidecar container to interact with the external snapshotter.
3. Please refer to the first link in the 'References' section below for detailed documentation.


## Configuring the Amazon EBS CSI plugin to use IAM roles for service accounts
1. https://docs.aws.amazon.com/eks/latest/userguide/csi-iam-role.html
2. Use the AWS managed IAM policy AmazonEBSCSIDriverPolicy

## To use the external EBS CSI driver, we need to create a new StorageClass based upon it
1. kubectl apply -f /init/storageclass.yml

## Create the Snapshots
1. Before taking a snapshot
    1. Create a VolumeSnapshotClass with the newly installed driver
    2. Apply a PVC claim with the newly installed StorageClass above.
    3. Create a deployment using this pvc, apply all the files in deploy folder
2. To take the snapshot
    1. As the pod is up and connected to a PersistentVolume (which is a EBS Volume).
    2. Apply the VolumeSnapshot file with class set to VolumeSnapshotClass created above, or apply the files in the snapshot folder
    3. A snapshot is created. We can see the snapshot on EC2 Volumes or using 'kubectl get volumesnapshotcontents
3. Restore the snapshot
    1. To restore a new deployment using the snapshot created above.
    2. First create a new PersistentVolumeClaim referencing the above snapshot (or delete the older pod and apply the files in the restore folder).
    3. Apply the PVC
    4. Now deploy the new pod using the above PVC , so that the new pod uses new PersistentVolumeClaim to get a PersistentVolume that contains the data restored from the snapshot.

# References
1. https://aws.amazon.com/blogs/containers/amazon-ebs-csi-driver-is-now-generally-available-in-amazon-eks-add-ons/
2. Service Map : https://miro.com/app/board/uXjVPfE957w=/
