# Eventing Game

## Deploy a Kubernetes platform on a local environment

Create a new Kubernetes cluster using `kind`.

```shell
kind create cluster --config platform/kind-config.yml
```

Deploy `kapp-controller` from the Carvel suite.

```shell
kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/download/v0.35.0/release.yml
```

Set up the cluster to handle Carvel packages.

```shell
kapp deploy -a package-setup -f platform/package-setup.yml
```

Using `kapp`, install the packages necessary to build the platform for running the game.

```shell
kapp deploy -a platform -f platform/packages
```

Finally, deploy a Redis instance.

```shell
kapp deploy -a redis -f platform/redis
```

## Deploy the application services

Deploy the application services part of the game system as follows.

```shell
kapp deploy -a eventing-game -f applications/app.yml
```

Using the `kapp` CLI, you can get the most relevant information about the deployment in a very convenient way:

```shell
kapp inspect -a eventing-game
```

When the application is up and running, you can open a browser window, navigate to the DNS name you specified previously,
and play the game.
