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

Creating a Pod for testing
```sh
kubectl apply -f https://raw.githubusercontent.com/janudabethmin/falco-setup/refs/heads/main/ubuntu-pod.yaml
```

Get into the terminal of the Pod
```sh
kubectl exec -it $(kubectl get pods -l app=ubuntu -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

Can keep this running to check whether a specific rule is triggered
```sh
kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco | grep 'Warning Grep private keys'
```

Running a command that will trigger some rules
```sh
find /root -name "id_rsa"
```

Installing Falco Talon
```sh
helm install falco-talon falcosecurity/falco-talon --namespace falco
```

