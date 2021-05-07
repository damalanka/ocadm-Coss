# ocadm-Coss
Another repo for OpenShift with Coss

Hello All Welcome back...

[root@master ~]# oc projects
You have access to the following projects and can switch between them with 'oc project <projectname>':
default
kube-public
kube-service-catalog
kube-system
logging
management-infra
* nginix
openshift
openshift-ansible-service-broker
openshift-infra
openshift-node
openshift-template-service-broker
openshift-web-console
Using project "nginix" on server "https://master.lab.example.com".
[root@master ~]#
  
[root@master ~]# oc get projects
NAME DISPLAY NAME STATUS
default Active
kube-public Active
kube-service-catalog Active
kube-system Active
logging Active
management-infra Active
nginix Active
openshift Active
openshift-ansible-service-broker Active
openshift-infra Active
openshift-node Active
openshift-template-service-broker Active
openshift-web-console Active
[root@master ~]#


1.Authenticating Users:-
-------------------------
Enable "HTPasswdPasswordIdentityProvider" as an identity provider method.
Ensure no other user should
Create three users
lene
joe
greg
anthony
to access oc gui and cli having password "zaldebro".
Use "HTPasswdPasswordIdentityProvider" as an identity provider method.
Ensure no other user should be allowed to access any oc projects.
User password information are stored under
/etc/origin/master/htpasswd

[root@master ~]# oc project default
Now using project "default" on server "https://master.lab.example.com".
[root@master ~]#

[root@master ~]# htpasswd -b /etc/origin/master/htpasswd greg zeldebra
Adding password for user greg
[root@master ~]#

[root@master ~]# htpasswd -b /etc/origin/master/htpasswd joe zladebra
Adding password for user joe
[root@master ~]#

[root@master ~]# htpasswd -b /etc/origin/master/htpasswd antony zaldebra
Adding password for user antony
[root@master ~]#

[root@master ~]# oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated
system:authenticated:oauth
cluster role "self-provisioner" removed: ["system:authenticated" "system:authenticated:oauth"]
[root@master ~]#

[root@master ~]# cat /etc/origin/master/htpasswd
admin:$apr1$4ZbKL26l$3eKL/6AQM8O94lRwTAu611
developer:$apr1$4ZbKL26l$3eKL/6AQM8O94lRwTAu611
greg:$apr1$ejG3tSj0$VWp/VWdvHTYq30wPxZ1bt/
joe:$apr1$4UWZMZDG$ZtX3YlTgDWsUkAKC1zSar1
antony:$apr1$ANGXSkC2$cI5tLmooTMWywGSBooCSM0
[root@master ~]#



2.Create a persisten Volume of "exam-registry-volume" using /OCP_registry
"exam-registry-volume" to be bound with a PVC "exam-registry-claim" for
substituting/replacing the default "docker-registry" volume.Project (Default)

Create PV an PVC with below steps.
-----------------------------------

[root@master ~]# cat docker-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
name: exam-registry-volume
spec:
capacity:
storage: 5Gi
accessModes:
- ReadWriteMany
nfs:
path: /OSE_registry
server: workstation.lab.example.com
persistentVolumeReclaimPolicy: Recycle
claimRef:
name: exam-registry-claim
namespace: default
[root@master ~]#

[root@master ~]# oc get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM
STORAGECLASS REASON AGE
etcd-vol2-volume 1G RWO Retain Bound
openshift-ansible-service-broker/etcd 1d
exam-registry-volume 5Gi RWX Recycle Bound default/exam-registry-claim
3m
registry-volume 40Gi RWX Retain Bound default/registry-claim
1d
[root@master ~]#

[root@master ~]# oc get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
exam-registry-claim Bound exam-registry-volume 5Gi RWX 2m
registry-claim Bound registry-volume 40Gi RWX 1d
[root@master ~]#

[root@master ~]# oc delete pvc exam-registry-claim
persistentvolumeclaim "exam-registry-claim" deleted
[root@master ~]#

[root@master ~]# oc delete pv exam-registry-volume
persistentvolume "exam-registry-volume" deleted
[root@master ~]#

[root@master ~]# oc get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM
STORAGECLASS REASON AGE
etcd-vol2-volume 1G RWO Retain Bound
openshift-ansible-service-broker/etcd 1d
registry-volume 40Gi RWX Retain Bound default/registry-claim
1d
[root@master ~]#

[root@master ~]# oc get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
registry-claim Bound registry-volume 40Gi RWX 1d
[root@master ~]#

[root@master ~]# oc create -f docker-pv.yml
persistentvolume "exam-registry-volume" created
[root@master ~]#

[root@master ~]# oc set volume dc/docker-registry --add --overwrite --name=exam-registry-volume -t pvc
--claim-name=exam-registry-claim --claim-mode="ReadWriteMany" --claim-size="5Gi"
persistentvolumeclaims/exam-registry-claim
info: deploymentconfigs "docker-registry" was not changed
[root@master ~]#

[root@master ~]# oc get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM
STORAGECLASS REASON AGE
etcd-vol2-volume 1G RWO Retain Bound
openshift-ansible-service-broker/etcd 1d
exam-registry-volume 5Gi RWX Recycle Bound default/exam-registry-claim
4m
registry-volume 40Gi RWX Retain Bound default/registry-claim
1d
[root@master ~]#

[root@master ~]# oc get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
exam-registry-claim Bound exam-registry-volume 5Gi RWX 14s
registry-claim Bound registry-volume 40Gi RWX 1d
[root@master ~]#
