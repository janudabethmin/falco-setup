# Intial Setup of Falco and Falco Sidekick

Adding Falco helm charts
```sh
kubectl create namespace falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update falcosecurity
```

Visit [Falco Rules Explorer](https://thomas.labarussias.fr/falco-rules-explorer/) for detailed view of falco rules.

Installing Falco and Falco Sidekick
```sh
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.webui.redis.storageEnabled=false \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803 
```
Adding custom falco rules to the initial Falco installation
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

Adding custom falco rules to the existing Falco installation
```sh
helm upgrade falco falcosecurity/falco --namespace falco \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.webui.redis.storageEnabled=false \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803 \
  --reuse-values \
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
> [!NOTE]
> Default username and password both for Falco Sidekick UI is `admin`

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
---

# Installing Atomic Red for Threat Simulation

Create a new namespace for Atomic Red Team
```sh
kubectl create namespace atomic-red
```

Deploying Atomic Red to the cluster
```sh
kubectl apply -f https://raw.githubusercontent.com/janudabethmin/falco-setup/refs/heads/main/atomic-red.yaml
```

Getting into the Atomic Red's terminal
```sh
kubectl exec -it -n atomic-red deploy/atomicred -- bash
```

We need to use PowerShell to run the Atomic Red's commands because it only supports Windows.
```sh
pwsh
```
Importing Atomic Red module
```pwsh
Import-Module "~/AtomicRedTeam/invoke-atomicredteam/Invoke-AtomicRedTeam.psd1" -Force
```
Getting details of a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004 -ShowDetails
```
Check the prerequisites for a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004 -GetPreReqs
```
Running a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004
```

---
# Setting Up Falco Talon

Installing Falco Talon
```sh
helm install falco-talon falcosecurity/falco-talon --namespace falco
```


