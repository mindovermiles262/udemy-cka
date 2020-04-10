# Manual Scheduling

[Docs: nodeName](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodename)

Use the `nodeName` selector in a Pod's `spec` to manually schedule a pod.

```
# manual-scheduling-pod.yml
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: node01
```

Then use kubectl to destroy/create the pod.

## Hotscheduling

How do you schedule a pod on a node once it's started? The same way the kube-scheduler does; make an API POST request.

```
# pod-binding.yml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

Convert the `pod-binding.yml` to JSON:

```
{
  "apiVersion": "v1",
  "kind": "Binding",
  "metadata": {
    "name": "nginx"
  },
  "target": {
    "apiVersion": "v1",
    "kind": "Node",
    "name": "node02"
  }
}
```

Then make the POST request:

```
$ curl -X POST \
  --header "Content-Type:application/json" \
  -- data '{ POD-BINDING-JSON }' \
  http://$K8S_SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
```


# Taints and Tolerations

[Docs: Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

* Tolerant pods can and will be scheduled on untainted nodes!
* Use `NodeAffinity` to schedule pods on specific nodes

## Taints

* Applied to NODE

```
kubectl taint nodes [NODE] key=value:taint-effect

kubectl taint nodes node01 app=db:NoSchedule
```

### Taint Effects

* `NoSchedule`
  * k8s will not schedule pod on tainted node
* `PreferNoSchedule`
  * k8s will TRY NOT TO schedule pod on tainted node, but not required.
* `NoExecute`
  * Taint node01 with `ks taint node node01 app=myapp:NoExecute`
  * Toleration added to podB
  * PodA is evicted (killed) and will be rescheduled on node02

```
------------  ------------
| [A]  [B] |  | [C]  [D] |
|          |  |          |
|  node01  |  |  node02  |
------------  ------------

[ Taint Set ]

----------    ---------------
|  [*B]  |    | [C] [D] [A] |
|        |    |             |
| node01 |    |    node02   |
----------    ---------------
```

## Tolerations

* Applied to PODS in `definition.yaml` file

```
apiVersion: v1
kind: Pod
metadata:
spec:
  containers: 
    - [ ... ]
  tolerations:
    - key: "app"
      operator: "Equals"
      value: "db"
      effect: "NoSchedule"
```

# Node Affinity

[Docs: Affinity and Anti-Affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)

* Applied to PODS to schedule on specific Nodes.
* Can be complex queries 
  * Example: "Run this pod on nodes labeled Medium or Large"

Use the `spec.affinity:` selector:

```
# deployment-nodeaffinity-red.yml

apiVersion: apps/v1
kind: Deployment
metadata:
spec:
  containers:
    - [ ... ]
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: color
              operator: In
              values:
                - "blue"
```

## Operator Values

* `In`
* `NotIn`
* `Exists`
* `DoesNotExist`
