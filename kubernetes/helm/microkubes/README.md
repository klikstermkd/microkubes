# Microkubes Helm Chart

This directory contains a Kubernetes chart to deploy [Microkubes](https://github.com/Microkubes/microkubes) cluster.
The helm chart implements all the relevant configuration parameters that can be configured through  a [value file](https://github.com/Microkubes/microkubes/blob/helm/kubernetes/helm/microkubes/values.yaml) or directly on command line. Also the helm chart supports deploying using an ingress to kubernetes clusters that have a configured ingress controller.

## Prerequisites Details

* Kubernetes 1.9
* Install [Helm](https://github.com/helm/helm/releases)
* Make sure that you have helm tiller running in your cluster, if not run `helm init`
* Setup correctly kubectl and kubeconfig setup before running helm.

## Deploy Microkubes on kubernetes cluster

To deploy Microkubes on kubernetes cluster with the release name `<release-name>` within namespace `<namespace-name>`:

```console
$ cd kubernetes/helm
$ helm dependency update microkubes/
$ helm install microkubes/ --namespace <namespace-name> --name <release-name> \
    --set postgresql.postgresUser=kong,postgresql.postgresPassword=<secretpassword>,postgresql.postgresDatabase=kong
```

**Note:** PostgreSQL user should be `kong` and database `kong` as well

## Configuration

Production deployment values can be set in kubernetes/helm/microkubes/values.yaml [here](https://github.com/Microkubes/microkubes/blob/helm/kubernetes/helm/microkubes/values.yaml).
This file contains variables that will be passed to the templates. All configurable values should be placed in this file.

Alternatively, you can specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

> **Tip**: You can use values file different from the default [values.yaml](https://github.com/Microkubes/microkubes/blob/helm/kubernetes/helm/microkubes/values.yaml) that specifies the values for the parameters by providing that file while installing the chart. For example:
```console
$ helm install --namespace <namespace-name> --name <release-name> -f microkubes/values-development.yaml microkubes/
```

## Secrets

Platfrom secrets are all Base64 encoded. Microkubes [keys](https://github.com/Microkubes/microkubes/tree/master/docker/keys) and MongoDB init [script](https://github.com/Microkubes/microkubes/blob/master/docker/mongo/create_db_objects.sh) are base64 encoded and put in [microkubes secrets](https://github.com/Microkubes/microkubes/blob/helm/kubernetes/helm/microkubes/templates/microkubes-secrets.yaml) template file which creates platform secrets.

If you want to generate new keys first run the script:
```console
./keys/create.sh
```

then encode the content:
```console
base64 keys/default.pub  >&2
```

The encoded content put in the [microkubes secrets](https://github.com/Microkubes/microkubes/blob/helm/kubernetes/helm/microkubes/templates/microkubes-secrets.yaml) template file.

## Cleanup

To remove the spawned pods you can run a simple `helm del --purge <release-name>`.

Helm will however preserve created persistent volume claims, to also remove them execute the commands below.

```console
$ release=<release-name>
$ namespace=<namespace-name>
$ helm del --purge $release
$ kubectl -n $namespace delete pvc -l release=$release
```



