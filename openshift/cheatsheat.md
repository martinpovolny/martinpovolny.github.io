# OpenShift CLI Cheatsheet

## Quick Orientation

```bash
oc status                              # deployments + pods + routes at a glance
oc get pods                            # list pods
oc get all                             # all resources in current project
oc describe pod/<podname>              # detailed info: volumes, secrets, env, SA
oc logs pod/<podname>                  # container logs
oc logs pod/<podname> -f               # follow logs
```

## Deployments

### Create from image

```bash
oc new-app --name my-app --docker-image quay.io/hello/hello-world:v1.0
oc new-app --name my-app --as-deployment-config=true --docker-image quay.io/hello/hello-world:v1.0
```

### Create from source (S2I)

```bash
oc new-app --as-deployment-config --name cool-app \
  --build-env npm_config_registry=http://npm.company.com/repository/nodejs \
  nodejs:12~https://github.com/martinpovolny/cool-app#feature-branch \
  --context-dir app
```

### Environment, resources, volumes

```bash
oc set env <depl> --from secret/<name> --prefix DB_
oc set env <depl> --from configmap/<name>
oc set resources <depl> --limits=cpu=500m,memory=256Mi --requests=cpu=100m,memory=128Mi
oc set volume <depl> --add -t pvc --claim-size=1G --mount-path /mnt/data
oc set volume <depl> --add --mount-path=/mnt/secrets --type=secret --secret <secret_name>
```

## ConfigMaps

```bash
oc create configmap <name> --from-literal=key=value --from-file=file
oc set volume <depl> --add -t configmap -m /mount/path --name vol --configmap-name=<cm>
oc set env <depl> --from configmap/<cm>
```

## ServiceAccounts & SCCs

```bash
oc create sa <sa-name>
oc adm policy add-scc-to-user anyuid -z <sa-name>
oc set sa <depl> <sa-name>
```

## Rollouts (DeploymentConfig)

```bash
oc set triggers dc/<name> --from-config --remove   # stop auto-triggers
oc rollout latest dc/<name>                         # manual rollout
oc set triggers dc/<name> --from-config             # re-enable
```

## Secrets & Pull Secrets

```bash
oc create secret generic quayio \
  --type kubernetes.io/dockerconfigjson \
  --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json
oc secrets link default <secret> --for pull
```

### Cross-project image access

```bash
oc policy add-role-to-user system:image-puller \
  system:serviceaccount:project-a:default --namespace=project-b
```

## Routes

### Edge route with self-signed cert

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 \
  -subj "/C=US/ST=Oregon/L=Portland/O=Company/OU=Org/CN=myhost.example.com"
oc create route edge ts-route \
  --hostname myhost.example.com --key key.pem --cert cert.pem --service <svc>
```

## RBAC & Groups

```bash
oc adm groups new <group> <user1> <user2>
oc adm policy add-role-to-group <role> <group>           # project-scoped
oc adm policy add-cluster-role-to-group <role> <group>   # cluster-scoped
oc get rolebindings -o wide
```

### Disable self-provisioning

```bash
oc adm policy remove-cluster-role-from-user self-provisioner system:authenticated:oauth
```

### Remove kubeadmin

```bash
oc delete secret kubeadmin -n kube-system
```

## htpasswd Identity Provider

```bash
htpasswd -c -B -b /tmp/htpasswd user password
oc create secret generic htpasswd-secret --from-file=htpasswd=/tmp/htpasswd -n openshift-config
oc edit oauth/cluster -n openshift-config
```

## Quotas

```bash
oc apply -f <quota-manifest.yaml>
oc edit quota/<quota-name>
```

## Build Hooks

```bash
oc set build-hook bc/<buildconfig> --post-commit --command -- <command>
```

## Taints

```bash
oc adm taint nodes <node-name> <key>-                      # remove
oc adm taint nodes <node-name> <key>=<value>:<effect>      # add
```
