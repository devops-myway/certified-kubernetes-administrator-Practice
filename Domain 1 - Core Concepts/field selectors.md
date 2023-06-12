https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/

##### Field Selectors

Field selectors let you select Kubernetes objects based on the value of one or more resource fields. Here are some examples of field selector queries:


This kubectl command selects all Pods for which the value of the status.phase field is Running
```sh
kubectl get pods --field-selector status.phase=Running

```

```sh
kubectl get ingress --field-selector foo.bar=baz

```
###### Supported operators
You can use the =, ==, and != operators with field selectors (= and == mean the same thing).

```sh
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always

```
##### Multiple resource types
```sh
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default

```
