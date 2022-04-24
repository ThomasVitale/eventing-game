# Deployment of the Eventing Game application

First, update the app.yml file with your values for the external DNS name, Redis hostname and password.

Then, open a Terminal window from the current folder and run the following command to deploy all the services
composing the Eventing Game application.

```shell
kubectl apply -f app.yml
```

Using the `kapp` CLI, you can get the most relevant information about the deployment in a very convenient way:

```shell
kapp inspect -a eventing-game
```

When the application is up and running, you can open a browser window, navigate to the DNS name you specified previously,
and play the game.
