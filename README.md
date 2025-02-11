Adding Falco helm charts
```sh
kubectl create namespace falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update falcosecurity
```

Installing Falco and Falco Sidekick with custom rules
```sh
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.webui.redis.storageEnabled=false \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803 \
  -f custom-rules.yaml
```

Watching the logs of Falco
```sh
kubectl logs -n falco -c falco -f -l app.kubernetes.io/name=falco
```

Portforwarding Falco Sidekick UI
```sh
kubectl port-forward -n falco svc/falco-falcosidekick-ui  2802:2802
```

Installing Falco Talon
```sh
helm install falco-talon falcosecurity/falco-talon --namespace falco
```
