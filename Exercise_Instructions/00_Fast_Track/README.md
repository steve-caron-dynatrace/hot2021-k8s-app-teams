## Fast - Instructions and commands only

### Exercise #1 Explore Your Environment

### Exercise #2 Set up automatic import of Kubernetes labels and annotations

```sh
$ kubectl get po -l product=sockshop --all-namespaces 
```

Let's look at the description of the `carts` pod in the `sockshop-dev` namespace. 

```sh
$ kubectl describe po -l app=carts -n sockshop-dev
```

#### Personal dashboard

- Go to <b>Menu -> Transactions and Services</b>
- In the filter text box <b>(1)</b>, select:
  -  `Tag` `[Kubernetes]product` `sockshop`
  -  `Tag` `[Kubernetes]stage` `prod`
- Click on the edit icon (pencil) next to the screen title (Services). <b>(2)</b> Change the title to: `SockShop Prod services` and click on the check mark to confirm. 
- Click on <b>Pin to dashboard</b> <b>(3)</b>

### Exercise #3 Custom Service naming rules for Kubernetes

Target Service naming convention :  `k8s-project-namespace-app Web Service Name`

Let's apply that configuration in Dynatrace!

- Go in <b>Settings -> Server-side service monitoring -> Service naming rules</b> and click <b>Add a new rule</b>
- Provide a name to the rule, for example : `Sock Shop service names`
- First, we want this rule to apply only to containerized processes running in Kubernetes. This is done by defining a condition.
  - In the conditions drop-down, select the property `"Kubernetes namespace"` and the condition `"exists"`
  - Click <b>Add condition</b>
  - Select the property `"Process group tags"` and the condition `"equals"`
    - Select the tag `[Kubernetes]product` and the tag value `sockshop`
- For the name format, we can enter free text and/or use placeholders.
  - Placeholders are in between brackets { } to distinguish them from free text
  - Enter this format : 
    - `k8s-{ProcessGroup:KubernetesNamespace}.{ProcessGroup:KubernetesContainerName} {ProcessGroup:Kubernetes:canary} {Service:WebServiceName}`

### Exercise #4 Playing with Management Zones

```sh
$ ./create-management-zones.sh
```

### Exercise #5 Set up alert notifications

```sh
$ ./create-alerting-profiles.sh
```

### Exercise #06 Performance problem detection

```sh
$ kubectl apply -f ../sockshop/manifests/scenarios/carts-dev-new-build.yml
```

```sh
$ kubectl rollout undo deployments carts --to-revision=1 -n sockshop-dev
```

### Exercise #7 Canary Deployment

```sh
$ ./deploy-carts-frontend-v2.sh
```
```sh
$ ./configure-v1-v2-traffic-management.sh
```


```sh
$ ./toggle-sockshop-promo-ff.sh
```
At the prompt, enter <b>1</b>.

```sh
$ ./toggle-sockshop-promo-ff.sh
```
At the prompt, enter <b>2</b>.

```sh
$ ./revert-to-v1-traffic-management.sh
```

### Exercise #8 Import Prometheus Metrics

```sh
$ kubectl apply -f ../sockshop/manifests/scenarios/carts-db-with-prometheus-exporter.yml
```

```sh
$ kubectl get po -l name=carts-db --all-namespaces -w
```

### Exercise #9 Managing your workload resource usage

```sh
$ kubectl apply -f ../hipstershop/paymentservice-new.yaml
```

```sh
$ kubectl apply -f ../hipstershop/paymentservice-fix.yaml
```

```sh
$ kubectl apply -f ../hipstershop/paymentservice-fix-the-fix.yaml
```

```sh
$ ./toggle-easytravel-resources-scenario-1.sh
```

At the prompt, enter `1` to enable the scenario.

```sh
$ kubectl apply -f  ../easytravel/compute-resources-quota.yaml 
```
```sh
$ kubectl delete po -l app=easytravel-backend -n easytravel
```

```sh
$ kubectl apply -f ../easytravel/compute-limitrange.yaml
$ kubectl delete rs -l app=easytravel-backend -n easytravel
$ kubectl get po -n easytravel -w
```

```sh
$ ./toggle-easytravel-resources-scenario-1.sh
```
At the prompt, enter `2` to disable the scenario

:arrow_up_small: [Back to overview](../README.md)
