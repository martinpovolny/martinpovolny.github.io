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
