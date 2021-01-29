# Creating deployments/deployments config

Using s2i image with a repo, branch, context dir and a build-env variable:
```
oc new-app --as-deployment-config --name cool-app \
> --build-env npm_config_registry=http://npm.company.com/repository/nodejs \
> nodejs:12~https://github.com/martinpovolny/cool-app#feature-branch \
> --context-dir app
```
