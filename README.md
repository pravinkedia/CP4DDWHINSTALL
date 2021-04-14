# CP4DDWHINSTALL

## Db2 Warehouse Installation steps

### Preparation of Storage class

https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_latest/cpd/install/portworx-storage-classes.html

#### DB2 RWX shared volumes for System Storage, backup storage, future load storage, and future diagnostic logs storage
--
  cat <<EOF | oc create -f -
  allowVolumeExpansion: true
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
   name: portworx-db2-rwx-sc
  parameters:
   io_profile: cms
   block_size: 4096b
   nfs_v4: "true"
   repl: "3"
   sharedv4: "true"
   priority_io: high
  provisioner: kubernetes.io/portworx-volume
  reclaimPolicy: Retain
  volumeBindingMode: Immediate
  EOF

#### - Ready the node

oc adm taint node node_name icp4data=dedicated_specifier:NoSchedule --overwrite

oc adm drain node_name

oc adm uncordon node_name

E.G:

oc adm taint node worker9.cp4d-swat.cp.fyre.ibm.com icp4data=database-db2wh:NoSchedule --overwrite

oc adm drain worker9.cp4d-swat.cp.fyre.ibm.com --delete-local-data --ignore-daemonsets

oc adm uncordon worker9.cp4d-swat.cp.fyre.ibm.com

#### - Label the node

oc label node node_name icp4data=dedicated_specifier --overwrite

E.G.

oc label node worker9.cp4d-swat.cp.fyre.ibm.com icp4data=database-db2wh

#### - Check the node

oc get node --show-labels

#### — Pre-checks

./cpd-cli adm --repo ./repo.yaml --assembly db2wh namespace cpdswat --storageclass portworx-db2-rwx-sc --cluster-pull-prefix Registry_from_cluster --ask-push-registry-credentials --latest-dependency

#### — Install

./cpd-cli install --repo ./repo.yaml --assembly db2wh --arch x86_64 --namespace cpdswat --storageclass portworx-db2-rwx-sc --transfer-image-to default-route-openshift-image-registry.apps.cp4d-swat.cp.fyre.ibm.com/cpdswat --cluster-pull-prefix image-registry.openshift-image-registry.svc:5000/cpdswat --latest-dependency --target-registry-password $(oc whoami -t) --target-registry-username $(oc whoami) --insecure-skip-tls-verify

[INFO] [2021-04-14 00:39:12-0520] Parsing custom YAML file ./repo.yaml

ibm.comibm.com

#### - Creating Portworx storage classes | IBM Cloud Pak for Data

If you decide to use Portworx as your storage option, Cloud Pak for Data requires the following storage classes. You can create them either manually or automatically
