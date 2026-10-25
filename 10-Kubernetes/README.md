# Kubernetes Commands & Practical Guide

A practical Kubernetes reference for common `kubectl` commands, essential YAML, and everyday troubleshooting. Each command includes a complete example and a short explanation.

> **Before you start:** You need a working Kubernetes cluster and `kubectl` configured to access it. Names such as `nginx`, `redis`, `httpd-frontend`, `dev`, and `docker-desktop` are examples; the referenced resource or context must exist for inspection or update commands to work. Commands in the tables are alternatives and individual operations, **not** a script to run top to bottom.

## Table of Contents

- [Cluster and Context](#cluster-and-context)
- [Pods and Containers](#pods-and-containers)
- [YAML and Resource Management](#yaml-and-resource-management)
- [ReplicaSets and Deployments](#replicasets-and-deployments)
- [Updates and Rollbacks](#updates-and-rollbacks)
- [Services and Networking](#services-and-networking)
- [Namespaces and Labels](#namespaces-and-labels)
- [ConfigMaps and Secrets](#configmaps-and-secrets)
- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [Practical YAML Examples](#practical-yaml-examples)

---

## Cluster and Context

| Task | Command | Explanation |
|---|---|---|
| Check kubectl version | `kubectl version --client` | `--client` shows the installed CLI version without requesting the server version. |
| Check cluster access | `kubectl cluster-info` | Displays the API server address and cluster service information. |
| List nodes | `kubectl get nodes` | Shows cluster nodes and their readiness. |
| List available contexts | `kubectl config get-contexts` | Lists saved cluster/user/namespace combinations. |
| See the current context | `kubectl config current-context` | Shows which context `kubectl` uses by default. |
| Switch context | `kubectl config use-context docker-desktop` | Switches to the named context; use a name actually listed by `get-contexts`. |

**Remember:** A *context* chooses your cluster connection and default namespace; it does not create a new cluster.

## Pods and Containers

| Task | Command | Explanation |
|---|---|---|
| List Pods | `kubectl get pods` | Lists Pods in the current namespace. |
| List Pods everywhere | `kubectl get pods -A` | `-A` includes **all namespaces**. |
| Show Pod IP and node | `kubectl get pods -o wide` | `-o wide` adds details such as IP address and assigned node. |
| Create an Nginx Pod | `kubectl run nginx --image=nginx` | Creates a Pod named `nginx`; `--image` selects the container image. |
| Inspect a Pod | `kubectl describe pod nginx` | Shows container details, image, state, conditions, and Events. |
| View complete Pod YAML | `kubectl get pod nginx -o yaml` | `-o yaml` prints the resource configuration and status as YAML. |
| Identify all container names | `kubectl get pod webapp -o jsonpath='{.spec.containers[*].name}'` | `-o jsonpath` selects fields; `[*]` includes **every defined container**, even one whose image cannot be pulled. |
| Identify container images | `kubectl get pod webapp -o jsonpath='{.spec.containers[*].image}'` | Prints the images defined for every container in the `webapp` Pod. |
| Read Pod logs | `kubectl logs nginx` | Shows logs from the Pod's default or only container. |
| Follow live logs | `kubectl logs -f nginx` | `-f` streams new log lines until stopped with `Ctrl+C`. |
| Choose a container for logs | `kubectl logs webapp -c nginx` | `-c nginx` selects the container named `nginx` in a multi-container Pod. |
| Open a container shell | `kubectl exec -it nginx -- sh` | `-it` starts an interactive terminal; `-- sh` runs the shell inside the container. |
| Watch Pod changes | `kubectl get pods -w` | `-w` keeps showing changes; stop with `Ctrl+C`. |
| Delete a Pod | `kubectl delete pod nginx` | Deletes the specified Pod. A controller may recreate its managed Pods. |

**Lab tip:** A Pod with two entries under `spec.containers` has **two defined containers**, even when one is stuck in `ImagePullBackOff`. The `Events` table lists events, **not** containers.

## YAML and Resource Management

| Task | Command | Explanation |
|---|---|---|
| Create or update a Pod from YAML | `kubectl apply -f redis-pod.yaml` | `-f` reads the local manifest file; `apply` creates or updates the resource. |
| Create or update a Deployment | `kubectl apply -f httpd-deployment.yaml` | Applies the Deployment definition in that file. |
| Preview YAML without creating it | `kubectl apply --dry-run=client -f httpd-deployment.yaml -o yaml` | `--dry-run=client` avoids submitting the change; `-o yaml` prints the prepared object. |
| Compare desired and live resources | `kubectl diff -f httpd-deployment.yaml` | Shows differences between the YAML and the cluster; exit status `1` means differences were found. |
| Find resources defined in a file | `kubectl get -f httpd-deployment.yaml` | Displays the object named inside the manifest. |
| Look up a YAML field | `kubectl explain deployment.spec.replicas` | Shows official field documentation in your cluster's API schema. |
| Edit a live Deployment | `kubectl edit deployment httpd-frontend` | Opens the live resource in an editor; does **not** update your local YAML file. |

**YAML rules:** Indentation uses spaces, `-` starts a list item, and keys such as `kind: Deployment` are case-sensitive. For Deployments, `selector.matchLabels` must match `template.metadata.labels`.

## ReplicaSets and Deployments

| Task | Command | Explanation |
|---|---|---|
| List ReplicaSets | `kubectl get rs` | Shows ReplicaSets and desired/current/ready replica counts. |
| Inspect matching ReplicaSets | `kubectl describe rs -l app=httpd-frontend` | `-l` selects ReplicaSets with the given label. |
| List Deployments | `kubectl get deployments` | Shows available and desired replica counts. |
| Create a basic Deployment | `kubectl create deployment web --image=nginx` | Creates Deployment `web` using the Nginx image. |
| Inspect a Deployment | `kubectl describe deployment httpd-frontend` | Shows its replicas, image, selector, conditions, and events. |
| Scale a Deployment | `kubectl scale deployment httpd-frontend --replicas=5` | `--replicas=5` changes the desired Pod count to five. |
| Delete a Deployment | `kubectl delete deployment web` | Deletes `web` and normally removes its managed ReplicaSet and Pods. |

**Relationship:** `Deployment -> ReplicaSet -> Pods -> Containers`. In normal application deployments, create a **Deployment** instead of managing ReplicaSets directly.

## Updates and Rollbacks

| Task | Command | Explanation |
|---|---|---|
| Update a Deployment image | `kubectl set image deployment/httpd-frontend httpd=httpd:2.4` | Changes container `httpd` to image `httpd:2.4`, triggering a rollout when the image differs. |
| Wait for rollout | `kubectl rollout status deployment/httpd-frontend` | Watches until the Deployment rollout completes or fails. |
| View rollout history | `kubectl rollout history deployment/httpd-frontend` | Lists recorded rollout revisions. |
| Undo the latest rollout | `kubectl rollout undo deployment/httpd-frontend` | Restores the previous Deployment Pod template revision. |
| Restart Deployment Pods | `kubectl rollout restart deployment/httpd-frontend` | Starts a rolling restart without manually deleting Pods. |
| Fix an existing Pod's image | `kubectl set image pod/redis redis=redis` | Changes the `redis` container image in Pod `redis` (useful when its original image was `redis123`). |

**Important:** A direct `set image` change does not edit a local YAML file. If you manage the resource with YAML, update the file as well to avoid configuration drift.

## Services and Networking

| Task | Command | Explanation |
|---|---|---|
| List Services | `kubectl get svc` | Shows Service names, types, ClusterIPs, and ports. |
| Create a ClusterIP Service | `kubectl expose deployment httpd-frontend --port=80 --target-port=80 --type=ClusterIP --name=httpd-service` | Exposes port `80` internally; `--target-port` is the container destination, `--type` is the Service type, and `--name` names it. |
| Create a NodePort Service | `kubectl expose deployment httpd-frontend --port=80 --target-port=80 --type=NodePort --name=httpd-nodeport` | Allocates a node port for reaching the app through a node address (if networking allows). |
| Create a LoadBalancer Service | `kubectl expose deployment httpd-frontend --port=80 --target-port=80 --type=LoadBalancer --name=httpd-lb` | Requests an external load balancer; actual provisioning depends on the cluster environment. |
| Inspect a Service | `kubectl describe svc httpd-service` | Displays selectors, ports, and backend endpoint information. |
| See Service backends | `kubectl get endpointslices -l kubernetes.io/service-name=httpd-service` | Lists EndpointSlices belonging to the named Service; useful when traffic is not reaching Pods. |
| Access a Deployment locally | `kubectl port-forward deployment/httpd-frontend 8080:80` | Forwards your machine's port `8080` to application port `80`; then visit `http://localhost:8080`. |
| Delete a Service | `kubectl delete svc httpd-nodeport` | Removes that Service without deleting the Deployment's Pods. |

**Service types:** `ClusterIP` = internal cluster access; `NodePort` = node IP plus allocated port; `LoadBalancer` = external balancer when supported. You can create `httpd-service` **either** with the `expose` command **or** with the Service YAML below, not both.

## Namespaces and Labels

| Task | Command | Explanation |
|---|---|---|
| List namespaces | `kubectl get namespaces` | Shows namespaces in the cluster. |
| Create a namespace | `kubectl create namespace dev` | Creates a namespace named `dev`. |
| Create a Pod in `dev` | `kubectl run nginx-dev --image=nginx -n dev` | `-n dev` creates the Pod in the `dev` namespace. |
| Show Pods in `dev` | `kubectl get pods -n dev` | Limits results to that namespace. |
| Show Pod labels | `kubectl get pods --show-labels` | `--show-labels` adds a column listing Pod labels. |
| Assign a label | `kubectl label pod nginx team=demo` | Adds label `team=demo` to the existing `nginx` Pod. |
| Filter by label | `kubectl get pods -l team=demo` | `-l` selects only Pods with that exact label. |
| Set default namespace | `kubectl config set-context --current --namespace=dev` | `--current` edits the active context, so future commands default to `dev`. |
| Restore default namespace | `kubectl config set-context --current --namespace=default` | Restores `default` for future commands using this context. |

**Warning:** `kubectl delete namespace dev` removes the namespace **and its resources**. Use it only when you intend to remove everything in `dev`.

## ConfigMaps and Secrets

| Task | Command | Explanation |
|---|---|---|
| Create a ConfigMap | `kubectl create configmap app-config --from-literal=APP_MODE=production` | Stores the non-sensitive key `APP_MODE` with value `production`; `--from-literal` sets a value directly. |
| List ConfigMaps | `kubectl get configmaps` | Lists ConfigMaps in the current namespace. |
| Inspect a ConfigMap | `kubectl describe configmap app-config` | Displays its keys and values. |
| Create a demo Secret | `kubectl create secret generic app-secret --from-literal=API_KEY=demo-only` | Creates an Opaque Secret with a **dummy** value; never type real secrets into shell history. |
| List Secrets | `kubectl get secrets` | Lists Secret metadata, not plaintext values. |
| Inspect Secret metadata | `kubectl describe secret app-secret` | Shows Secret type and key names, not secret values. |
| Delete a demo Secret | `kubectl delete secret app-secret` | Removes the named Secret. |

**Keep credentials safe:** Kubernetes Secrets are not automatically encrypted at rest in every cluster. Never commit real credentials, passwords, or tokens into a public GitHub repository.

## Monitoring and Troubleshooting

| Task | Command | Explanation |
|---|---|---|
| View events in time order | `kubectl get events --sort-by=.metadata.creationTimestamp` | `--sort-by` sorts Events by creation timestamp. |
| Show Pod CPU and memory | `kubectl top pods` | Uses the metrics API; requires a running metrics provider such as Metrics Server. |
| Show node CPU and memory | `kubectl top nodes` | Displays node resource usage when metrics are available. |
| Find Pending Pods | `kubectl get pods --field-selector=status.phase=Pending` | `--field-selector` filters resources by an API field. |
| Check a Pod's phase | `kubectl get pod redis -o jsonpath='{.status.phase}'` | Prints `Pending`, `Running`, `Failed`, etc.; this is **not** the same as the container's waiting reason. |
| Read previous container logs | `kubectl logs nginx --previous` | `--previous` shows logs from the previous instance of a restarted container, when available. |

### Common lab problems

| Problem | Why it happens | What to check or do |
|---|---|---|
| `ErrImagePull` / `ImagePullBackOff` | The image name/tag may be invalid, private, or unavailable. | `kubectl describe pod redis`; if the container is named `redis` and its image is `redis123`, use `kubectl set image pod/redis redis=redis`. |
| Two containers, but only one running | A Pod can define multiple containers even when one image fails. | `kubectl get pod webapp -o jsonpath='{.spec.containers[*].name}'`; count the names, **not** the Events rows. |
| `Deployment` YAML rejected | A wrong `kind` capitalization or mismatched selector and Pod labels can invalidate it. | Use `kind: Deployment` and make `selector.matchLabels` equal `template.metadata.labels`. |
| `Pending` Pod | Scheduling, storage, image, or other dependencies may not be ready. | Start with `kubectl describe pod redis` and read its Events. |
| Service has no reachable backends | Its selector may not match any ready Pods. | Compare `kubectl describe svc httpd-service` with `kubectl get pods -l app=httpd-frontend`. |
| YAML edited but live resource unchanged | Editing a local file does not automatically apply it to the cluster. | Run `kubectl apply -f redis-pod.yaml`, then verify with `kubectl get pod redis -o yaml`. |

## Practical YAML Examples

Save these YAML snippets in files with the specified names. They can be used with the `kubectl apply -f ...` commands shown earlier. All three examples use the **default namespace** unless you specify another one.

### Pod: `redis-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis
```

The `-` starts one container entry. The container's `name` and `image` are properties of the **same** entry.

### Deployment: `httpd-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd-frontend
  template:
    metadata:
      labels:
        app: httpd-frontend
    spec:
      containers:
        - name: httpd
          image: httpd:2.4-alpine
          ports:
            - containerPort: 80
```

- `replicas: 3` keeps three Pods available when possible.
- `selector.matchLabels` **must match** `template.metadata.labels`.
- The first `spec` configures the **Deployment**; `template.spec` configures the **Pods**.
- Each Pod runs one `httpd` container using `httpd:2.4-alpine`.

### Service: `httpd-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
spec:
  type: ClusterIP
  selector:
    app: httpd-frontend
  ports:
    - port: 80
      targetPort: 80
```

The Service selects Pods labeled `app: httpd-frontend` and forwards port `80` to port `80` on those Pods.

### Verify the Deployment and Service

```bash
kubectl apply -f httpd-deployment.yaml
kubectl apply -f httpd-service.yaml
kubectl get deployments
kubectl get rs
kubectl get pods -l app=httpd-frontend
kubectl get svc httpd-service
```

**Do not run both** the `kubectl expose ... --name=httpd-service` command and `kubectl apply -f httpd-service.yaml` for the same Service as separate creation methods. Choose one method and use it consistently.

---

**Official references:** [kubectl Quick Reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/) | [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) | [Services](https://kubernetes.io/docs/concepts/services-networking/service/) | [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
