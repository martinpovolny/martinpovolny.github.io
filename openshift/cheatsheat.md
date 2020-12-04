# OpenShift command-line cheatsheet

### Quick orientation in a project

 * `oc status` -- displays deployments/demployment configs + pods + routes, exposed routes
next
 *`oc get pods` -- get the list of pods
 * `oc describe pod/<podname>` -- detailed info about a pod, volumes missing? secrets missing? env variables, SA, etc.
 * `oc logs pod/<podname>` -- get logs from a process in the pod


### Basic `oc` commands for dealing with Deployments and Deployment config

 * `oc set env --from <e.g. secret> --prefix ... <depl>` -- common to share password for a database etc.
 * `oc set sa` -- setting a SA to e.g. give the `anyuid` scc, needed when a process tries to run as root
 * `oc set resources`
 * `oc set volume --add .... -t pvc --claim-size=1G <depl> --mount-path -- add ...` a volume to a dc or deployment and mount it

### Create, setup, add a SA (e.g. to fix a deployment needing a process running as root)
 1. `oc create sa <sa name>`
 1. `oc adm policy add-scc-to-user anyuid -z <sa name>`
 1. `oc set sa` ...

### Groups etc.
 * `oc adm groups new <group name> <user1> <user2....>`
 * `oc adm policy add-role-to-group <role> group` -- add per project role
 * ... `add-cluster-role-to` -- add cluster/global role
 * Disable self-provisioning `oc adm policy remove-cluster-role-from-user self-provisioner system:authenticated:oauth`
Details: https://docs.openshift.com/container-platform/3.6/admin_solutions/user_role_mgmt.html (toto: where's 4.6?)

#### Remove kubeadmin

https://docs.openshift.com/container-platform/4.6/authentication/remove-kubeadmin.html

`oc delete secrets kubeadmin -n kube-system`

### Setup htpasswd auth for a cluster
 * `htpasswd -c -B -b user password` ...
 * `oc create secret generic <sec.name> --from-file=htpasswd=<file path> -n openshift_config`
 * `oc edit -n openshift_config oauth/cluster`
 * follow https://docs.openshift.com/container-platform/4.6/authentication/identity_providers/configuring-htpasswd-identity-provider.html (Sample HTPasswd CR)

### Quota
 * follow https://docs.openshift.com/container-platform/4.6/applications/quotas/quotas-setting-per-project.html (Sample resource quota definitions)
 * `oc apply -f <manifest>`
 * `oc edit quota/<quota_name>`
