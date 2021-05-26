# Creating deployments/deployments config

Using s2i image with a repo, branch, context dir and a build-env variable:
```
oc new-app --as-deployment-config --name cool-app \
> --build-env npm_config_registry=http://npm.company.com/repository/nodejs \
> nodejs:12~https://github.com/martinpovolny/cool-app#feature-branch \
> --context-dir app
```


# Configmaps

Creating similar to secrets:

```oc create configmap <name> --from-literal=key=value --from-file=file```

Setting to Deployment/DC same as secrets:

```oc set volume <dc> --add -t configmap -m </mount/path> --name <vol.name> --configmap-name=<cm> ```

```oc set env <depl/dc> --from comfigmap/<my_cm>```

# Rollouts

Stop:
```oc set triggers dc/dc_name --from-config --remove```

Manual:
```oc rollout latest <dc_name>```

Start again:
```oc set triggers dc/dc_name --from-config```

# Pull secrets

```
oc secrets greate generic --type kubernetes.io/dockerconfigjson -from-file=.dockercfg=${XDG_RUNTIME_DIR}/containers/auth.json
oc secrets link default <secret> --for pull
```

# Allowing access cross project

See https://docs.openshift.com/container-platform/4.5/openshift_images/managing_images/using-image-pull-secrets.html

Access for pods in project-a to access project-b's images:

```
oc policy add-role-to-user system:image-puller system:serviceaccount:project-a:default --namespace=project-b
```

Access for any SA in project-a to access project-b's images:
```
oc policy add-role-to-group system:image-puller system:serviceaccounts:project-a --namespace=project-b
```
