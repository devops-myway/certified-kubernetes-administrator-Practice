https://stackoverflow.com/questions/58764509/why-container-object-in-pod-yaml-file-has-list-value-rather-than-a-map-value

##### Validation errors that one may encounter in production

error validating data:

- ValidationError(ScaledJob.spec.jobTargetRef.template.spec.containers): invalid type for sh.keda.v1alpha1.ScaledJob.spec.jobTargetRef.template.spec.containers: got "map", expected "array",

- ValidationError(ScaledJob.spec): unknown field "revisionHistoryLimit" in sh.keda.v1alpha1.ScaledJob.spec,

- ValidationError(ScaledJob.spec): unknown field "selector" in sh.keda.v1alpha1.ScaledJob.spec,

- ValidationError(ScaledJob.spec): unknown field "strategy" in sh.keda.v1alpha1.ScaledJob.spec]

##### Pod is capable of running multiple containers. That's the reason containers object is a list or arrays instead of map.
Lists (aka Arrays) are used when we need to provide a List (or Array) of objects/elements of the same type like in case of containers in Deployment definition:
container element is an array as it may contain many objects/elements/items of the same type ( in this case containers ) which can have same properties.
In container section you can define many containers and each of them will have its unique name, image and ports. - sign indicates one element or item of the array

``````sh
kind: Pod
...
spec:
  containers:
  - name: busybox
    image: busybox:latest
  - name: nginx
    image: nginx:1.7.9
  - name: redis
    image: redis:latest
    ports:
    - containerPort: 80
    
--
  containers:
  - name: nginx
    image: nginx:1.7.9
  - image: redis
    name: redis
  - command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    image: busybox
    name: myapp-container

``````
##### Maps (aka Dictionaries)
Maps (aka Dictionaries) are used when providing a set of key: value pairs. e.g. you provide different set of labels in metadata section of your Deployment definition.
metadata element is also in a form of Dictionary as it contains a set of unique keys ( name and labels in this particular case).

let's take a look at the spec section. Note that all three keys that are children of the spec element,
namely replicas, selector and template, are unique in the whole set and that's the reason they are in a form of a Dictionary ( or a Map ).

``````sh
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  labels:
    app: nginx
    type: front-end

-- .spec:

spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx

``````