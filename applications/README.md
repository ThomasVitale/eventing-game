# Deployment of the Eventing Game application

First, deploy the Game UI application.

```shell
kubectl apply -k game-ui
```

Then, deploy the Game Controller application.

```shell
kubectl apply -k game-controller
```

Finally, deploy the Level 1 application.

```shell
kubectl apply -k level-1
```
