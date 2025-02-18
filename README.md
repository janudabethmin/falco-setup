# Setting Up Falco Talon

> [!CAUTION]
> **Falco Talon** needs to be installed before **Falco** and **Falco Sidekick**

### Download Falco Talon chart to local
```sh
git clone https://github.com/falcosecurity/charts.git
```

### Remove the existing default rules and add custom ones
```sh
cd charts/charts/falco-talon
```

```sh
rm rules.yaml
```

```sh
wget https://raw.githubusercontent.com/janudabethmin/falco-setup/refs/heads/main/rules.yaml 
```

### Install falco talon

> [!IMPORTANT]
> Remember to be in the charts/charts/falco-talon directory before running the command below.

```sh
helm upgrade --install falco-talon -n falco --create-namespace .
```

---

# Setup of Falco and Falco Sidekick

### Adding Falco helm charts
```sh
kubectl create namespace falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update falcosecurity
```

Visit [Falco Rules Explorer](https://thomas.labarussias.fr/falco-rules-explorer/) for detailed view of falco rules.

### Installing Falco and Falco Sidekick
```sh
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.webui.redis.storageEnabled=false \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803 
```

### Adding custom falco rules to the initial Falco installation
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

### Adding custom falco rules to the existing Falco installation
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

> [!IMPORTANT]
> The custom rule that we are adding here is for a attomic red test that will not be identified by Falco default rules. Will be described in the next section.

### Watching the logs of Falco
```sh
kubectl logs -n falco -c falco -f -l app.kubernetes.io/name=falco
```

### Portforwarding Falco Sidekick UI
```sh
kubectl port-forward -n falco svc/falco-falcosidekick-ui  2802:2802
```
> [!NOTE]
> Default username and password both for Falco Sidekick UI is `admin`

### Creating a Pod for testing
```sh
kubectl apply -f https://raw.githubusercontent.com/janudabethmin/falco-setup/refs/heads/main/ubuntu-pod.yaml
```

### Get into the terminal of the Pod
```sh
kubectl exec -it $(kubectl get pods -l app=ubuntu -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

### Can keep this running to check whether a specific rule is triggered
```sh
kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco | grep 'Warning Grep private keys'
```

### Running a command that will trigger some rules
```sh
find /root -name "id_rsa"
```
---

# Installing Atomic Red for Threat Simulation

### Create a new namespace for Atomic Red Team
```sh
kubectl create namespace atomic-red
```

### Deploying Atomic Red to the cluster
```sh
kubectl apply -f https://raw.githubusercontent.com/janudabethmin/falco-setup/refs/heads/main/atomic-red.yaml
```

### Getting into the Atomic Red's terminal
```sh
kubectl exec -it -n atomic-red deploy/atomicred -- bash
```

### We need to use PowerShell to run the Atomic Red's commands because it only supports Windows.
```sh
pwsh
```
### Importing Atomic Red module
```pwsh
Import-Module "~/AtomicRedTeam/invoke-atomicredteam/Invoke-AtomicRedTeam.psd1" -Force
```
### Getting details of a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004 -ShowDetails
```
### Check the prerequisites for a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004 -GetPreReqs
```
### Running a specific atomic red command
```pwsh
Invoke-AtomicTest T1070.004
```

### Tables of Atomic Red Team Attacks That Can Be Used to Test Falco Rules

| Attack    | Command to View Logs | Grep the specific logs using pipes | Command to Execute Attack | Description | Identified by Default Falco Rules? |
|-----------|----------------------|---------------------------|-------------|------------------------------------|------------------|
| T1070.004 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Bulk data has been removed from disk'``` | ```Invoke-AtomicTest T1070.004``` | Bulk file deletion | Yes |
| T1556.003 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Sensitive file opened for reading by non-trusted program'``` | ```Invoke-AtomicTest T1556.003``` | Modify Authentication Process | Yes |
| T1036.005 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Executing binary not part of base'``` | ```Invoke-AtomicTest T1036.005``` | Masquerading: Match Legitimate Name or Location | Yes |
| T1070.002 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Log files were tampered'``` | ```Invoke-AtomicTest T1070.002``` | Indicator Removal on Host | Yes |
| T1070.003 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Shell history had been deleted or renamed'``` | ```Invoke-AtomicTest T1070.003``` | Clear Command History | Yes |
| T1014 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Linux Kernel Module injection from container detected'` | `Invoke-AtomicTest T1014` | Kernel Module Based Rootkit | Yes |
| T1037.004 | ```kubectl logs -f --tail=0 -n falco -c falco -l app.kubernetes.io/name=falco``` | ```grep 'Potentially malicious Python script'``` | ```Invoke-AtomicTest T1037.004``` | Boot Initialization - RC Scripts | No |

> [!IMPORTANT]
> We have added the custom rule for the T1070.004 attack to the custom-rules.yaml file. Use the commands in the previous section to add the custom rules to Falco.

### More commands to try that will trigger alerts

```sh
find /root -name "id_rsa"
```

```sh
cat /etc/shadow > /dev/null
```

```sh
grep "aws_secret_access_key" /etc/shadow
```

---

# Testing Falco Talon

### Get into the terminal of the Ubuntu Pod created before
```sh
kubectl exec -it $(kubectl get pods -l app=ubuntu -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

### Running a command
```sh
find /root -name "id_rsa"
```

> [!NOTE]
> This will add a tag ***suspicious: "true"*** to the Ubuntu Pod as a responce to the detection done by falco, as we wrote in the falco-talon rules. 