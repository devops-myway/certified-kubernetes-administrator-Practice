https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition

##### CustomResourceDefinitions (CRDs)
A resource in the Kubernetes API is an endpoint that persists a collection of a particular type of object.
For example, several built-in objects, like pods and deployments, are exposed via an endpoint, and the API server manages their lifecycle.
Using CRD on Kubernetes, you are free to define, create, and persist any custom object.


```sh
cat << EOF | kubectl apply -f -
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.kplabs.internal
spec:
  group: kplabs.internal
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: number
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
EOF
```
Then a new namespaced RESTful API endpoint is created at: /apis/stable.example.com/v1/namespaces/*/crontabs/...

###### Create custom objects 
After the CustomResourceDefinition object has been created, you can create custom objects
```sh
cat << EOF | kubectl apply -f -
apiVersion: "kplabs.internal/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 3
EOF
--
kubectl get crontab
kubectl get ct -o yaml   # You should see that it contains the custom cronSpec and image fields from the YAML you used to create it:
```
AWS Service Operator Manifest File:

https://raw.githubusercontent.com/awslabs/aws-service-operator/master/configs/aws-service-operator.yaml


###### Delete a CustomResourceDefinition
``````sh
kubectl delete -f resourcedefinition.yaml
kubectl get crontabs

``````