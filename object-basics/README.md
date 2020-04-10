# Kubernetes Objects

## Pods

* Basic unit of deployment in k8s
* Collection (1+) of containers
* Single instance of an application

```
$ kubectl get pods
```

## Controllers

* Monitor k8s objects and respond appropriately
* Uses:
  * High Availability
  * Load Balancing and Scaling


### Replication Controller

* Older technology, replaced by ReplicaSet
* Starts and manages a specified group of pods

### ReplicaSet

* Same as Replication Controller except:
  * `apiVersion` is `apps/v1`
  * Must include `spec.selector` parameter:

```
spec:
  template: xxx
  replicas: 3
  selector:
    name: myapp-prod
```

* ReplicaSet can manage pods it did not create.

## Deployments

## Services
