Adding Falco helm charts
```sh
kubectl create namespace falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update falcosecurity
```

Installing Falco Talon
```sh
helm install falco falcosecurity/falco --namespace falco
```

Installing Falco and Falco Sidekick
```sh
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803
```
