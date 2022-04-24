# Setting up a Kubernetes platform on DigitalOcean

This guide explains how to set up a Kubernetes platform on DigitalOcean where to run the eventing game application.

## Installing the DigitalOcean CLI

Install the DigitalOcean CLI (`doctl`) as explained in the [official documentation](https://docs.digitalocean.com/reference/doctl/how-to/install/).
On macOS and Linux, you can use Homebrew.

```shell
brew install doctl
```

Following the documentation, create an API token to get access to Digital Ocean from `doctl`.

## Creating a Kubernetes cluster

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

## Configuring DNS

Make sure your custom domain name is configured to use the name servers by Digital Ocean.
Afterwards, register the domain name within Digital Ocean.

```shell
doctl compute domain create <domain>
```

## Installing Cert Manager

Let's start by installing Cert Manager as follows.

```shell
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.2/cert-manager.yaml
```

Then, navigate to `platform`, and run the following to configure a `ClusterIssuer` resource. Make sure you replace the email in the
resource with your own first.

```shell
kubectl apply -k cert-manager
```

## Installing Knative

One way to install and manage Knative is via the official [Operator](https://knative.dev/docs/install/operator/knative-with-operators/).

Install the Knative Operator as follows.

```shell
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.4.0/operator.yaml
```

You can verify the Operator is deployed with this command.

```shell
kubectl get deployment knative-operator
```

You can track the logs from the Operator as follows.

```shell
kubectl logs -f deploy/knative-operator
```

Next, install Knative Serving by running the following command. First, make sure you add your custom domain name in the
`KnativeServing` resource.

```shell
kubectl apply -k knative-serving
```

Check the deployment is completed successfully.

```shell
kubectl get deployment -n knative-serving
```

Finally, you can find the external IP address exposed by the load balancer configured for the Kourier Ingress used by Knative.

```shell
kubectl --namespace knative-serving get service kourier
```

Next, install Knative Eventing by running the following command.

```shell
kubectl apply -k knative-eventing
```

Check the deployment is completed successfully.

```shell
kubectl get deployment -n knative-eventing
```

## Installing Metrics Server

Open a Terminal window, navigate to `platform`, and run the following to install Metrics Server.

```shell
kubectl apply -k metrics-server
```

## Installing Redis

DigitalOcean offers a managed Redis Service. Create a Redis cluster as follows. Remember to replace `<your_region>` with the
geographical region you'd like to use. It should be the same as you used for the Kubernetes cluster. In my case, it's `ams3`.

```shell
doctl databases create game-redis \
    --engine redis \
    --region <your_region> \
    --version 6
```

The Redis server provisioning will take several minutes. You can verify the installation status with the following command. When it's `online`, then your Redis server is ready. Take note of the Redis resource ID. You'll need it later.

```shell
$ doctl databases list

ID               Name          Engine    Version    Region    Status
<redis-db-id>    game-redis    redis     6          ams3      creating
```

You can retrieve the details for connecting to Redis. Remember to replace `<redis-db-id>` with your Redis resource ID.

```shell
$ doctl databases connection <redis-db-id> --format Host,Port,User,Password

Host            Port            User            Password
<redis-host>    <redis-port>    <redis-user>    <redis-password>
```

Finally, create a Secret in the Kubernetes cluster with the Redis credentials that we'll pass the applications using Redis.
Populate the Secret with the information returned by the previous `doctl` command.

```shell
$ kubectl create secret generic polar-redis-credentials \
--from-literal=spring.redis.host=<redis_host> \
--from-literal=spring.redis.port=<redis_port> \
--from-literal=spring.redis.password=<redis_password> \
--from-literal=spring.redis.ssl=true
```
