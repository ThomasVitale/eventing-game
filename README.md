# Eventing Game

## Deploy a Kubernetes platform

This section will describe how to set up a Kubernetes platform locally with `kind` and in the public cloud on DigitalOcean.

Prerequisites:

* [kind](https://kind.sigs.k8s.io)
* [kapp](https://carvel.dev/kapp/)

### Local environment

Create a new Kubernetes cluster using `kind`.

```shell
kind create cluster --config local/kind-config.yml
```

Deploy `kapp-controller` from the Carvel suite.

```shell
kapp deploy -a kapp-controller -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/download/v0.36.1/release.yml --yes
```

Set up the cluster to handle Carvel packages.

```shell
kapp deploy -a cluster-setup -f local/cluster-setup --yes
```

Using `kapp`, install the packages necessary to build the platform for running the game.

```shell
kapp deploy -a platform-setup -f local/platform-setup --yes
```

Finally, deploy a Redis instance and an in-memory event broker.

```shell
kapp deploy -a data-packages -f local/data-packages --yes
```

### DigitalOcean

This section will describe how to set up a Kubernetes platform on DigitalOcean.

#### Set up a Kubernetes cluster

Install the DigitalOcean CLI (`doctl`) as explained in the [official documentation](https://docs.digitalocean.com/reference/doctl/how-to/install/).
On macOS and Linux, you can use Homebrew.

```shell
brew install doctl
```

Create a Kubernetes cluster using the Digital Ocean managed service with your choice of node size and region.

```shell
doctl k8s cluster create game-cluster \
  --node-pool "name=basicnp;size=s-2vcpu-4gb;count=3;label=type=basic;" \
  --region ams3
```

You can get a list of supported regions with the following command.

```shell
doctl k8s options regions
```

Regarding node sizes, you can get a complete list and information about prices with the following commands.

```shell
doctl k8s options sizes
doctl compute size list
```

You can even choose a specific Kubernetes version among the supported ones returned by the following command.

```shell
doctl k8s options versions
```

After creating the cluster, you can check it was correctly created with the following command.

```shell
doctl k8s cluster list
```

Also, verify your Kubernetes CLI is configured to work with the new cluster.

```shell
kubectl config current-context
```

#### Set up the platform

Deploy `kapp-controller` from the Carvel suite.

```shell
kapp deploy -a kapp-controller -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/download/v0.36.1/release.yml --yes
```

Set up the cluster to handle Carvel packages.

```shell
kapp deploy -a cluster-setup -f cloud/cluster-setup --yes
```

Check the IP address of the load balancer provisioned by Digital Ocean.

```shell
kubectl get service envoy -n contour-external
```

Then, update the IP address in `cloud/platform-setup/knative-serving.yml`.

Next, install the packages necessary to build the platform for running the game.

```shell
kapp deploy -a platform-setup -f cloud/platform-setup --yes
```

Finally, deploy a Redis instance and an in-memory event broker.

```shell
kapp deploy -a data-packages -f cloud/data-packages --yes
```

## Deploy the application services

Deploy the application services part of the game system as follows, after replacing `<conf-name>` with `devoxxuk`, 
`kubeconeu`, or `springio`.

```shell
kapp deploy -a eventing-game -f applications/<conf-name>/app.yml --yes
```

Using the `kapp` CLI, you can get the most relevant information about the deployment in a very convenient way:

```shell
kapp inspect -a eventing-game
```

When the application is up and running, you can open a browser window, navigate to the DNS name you specified previously,
and play the game.

## Delete the local cluster

Run this command to delete the local `kind` cluster.

```shell
kind delete cluster --name game-cluster
```
