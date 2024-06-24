This PR introduces support for updating container images in Plugin type applications with the imperative argocd write-back method.

## Key Features:

Environment Variable Modification: Instead of adding parameters to the helm source of the application manifest, this PR modifies the HELM_ARGS environment variable within the plugin source.

Integration with argocd-vault-plugin: The HELM_ARGS env variable is primarily used by argocd-vault-plugin to pass arguments to the helm template command before it generates the final manifest with vault values. For more details, visit https://argocd-vault-plugin.readthedocs.io/en/stable/usage/#with-additional-helm-arguments

## Example of my use case:

```yaml
annotations:
    argocd-image-updater.argoproj.io/image-list: php=registry.myapp.cloud/myapp/php:dev, nginx=registry.myapp.cloud/myapp/nginx:dev
    argocd-image-updater.argoproj.io/php.helm.image-tag: php.image.tag
    argocd-image-updater.argoproj.io/nginx.helm.image-tag: nginx.image.tag
    argocd-image-updater.argoproj.io/update-strategy: digest
```

Every time a commit is pushed to the dev branch, images are built with the dev tag in the CI pipeline.

The dev tag is mutable, which is why the digest strategy is used.

php.image.tag and nginx.image.tag are keys inside the values.yaml file that need to be overridden if a new digest of the images tag exists.

With auto-sync enabled, the application is deployed automatically.

## Initial manifest:

```yaml
source:
  plugin:
    env:
      - name: HELM_ARGS
        value: >-
          -f ../../dev/charts/myapp/values.yaml
```

## Updated manifest after an image update:

```yaml
source:
  plugin:
    env:
      - name: HELM_ARGS
        value: >-
          -f ../../dev/charts/myapp/values.yaml 
          --set php.image.tag=dev@sha256:fb4b84725901d2c7214e080f2314e55dbc9c2cf2b7a9845c288c3014c7d614ab
          --set nginx.image.tag=dev@sha256:1e6a0957539cfb89a691ee721d4cd23ec5f4a9870c674fe4e8a50896d9ba386e
```

All previously defined arguments are preserved.

If a --set argument is already defined, it will be modified; otherwise, it will be added.

How to try it:

You can try out this feature by applying the following command:

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/rpg600/argocd-image-updater/plugin-type-support-wmanifest/manifests/install.yaml
```
