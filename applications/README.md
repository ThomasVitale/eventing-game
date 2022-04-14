# Deployment of the Eventing Game application

First, deploy the Game UI application.

```shell
kubectl apply -k game-frontend
```

Then, deploy the various functions composing the system.

```shell
kubectl apply -k start-game
```

```shell
kubectl apply -k start-level
```

```shell
kubectl apply -k level-1
```
