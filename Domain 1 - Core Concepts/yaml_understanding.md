#### List
- A list is a collection of items under a key, each prefixed with a dash "-".

```sh
# Simple list of strings
fruits:
  - apple
  - banana
  - cherry

# Inline list (same thing, compact form)
fruits: [apple, banana, cherry]

```
####  YAML Dictionaries (Maps)
- A dictionary is a set of key-value pairs nested under a parent key.
```sh
# Dictionary
person:
  name: John
  age: 30
  city: New York

# Inline dictionary (same thing, compact form)
person: {name: John, age: 30, city: New York}

```
####  List of Dictionaries
This is the most common pattern in Kubernetes — a list where each item is a dictionary.
```sh
employees:
  - name: Alice       # item 1 (a dictionary)
    role: engineer
    age: 28

  - name: Bob         # item 2 (a dictionary)
    role: manager
    age: 35

```
####  Nested Dictionaries inside Dictionaries
- Dictionaries can be infinitely nested.

```sh
server:
  database:
    host: localhost
    port: 5432
    credentials:
      username: admin
      password: secret

```
####  Real Kubernetes Examples
Pod Manifest — spot all 3 patterns:
```sh
apiVersion: v1              # scalar
kind: Pod                   # scalar

metadata:                   # DICTIONARY starts here
  name: my-pod              #   scalar inside dict
  labels:                   #   DICTIONARY inside dict
    app: my-app             #     scalar
    env: production         #     scalar

spec:                       # DICTIONARY
  containers:               # LIST OF DICTIONARIES starts here
    - name: nginx           #   dict item 1
      image: nginx:1.21     #
      ports:                #   LIST OF DICTIONARIES (nested)
        - containerPort: 80 #     dict item 1
        - containerPort: 443#     dict item 2
      env:                  #   LIST OF DICTIONARIES (nested)
        - name: ENV_MODE    #     dict item 1
          value: production #
        - name: LOG_LEVEL   #     dict item 2
          value: info       #

    - name: sidecar         #   dict item 2 (second container)
      image: busybox        #
```
####  ConfigMap — Dictionary of key-value data:

```sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config          # scalar
data:                       # DICTIONARY
  database_url: postgres://localhost:5432
  log_level: debug
  allowed_hosts: "*"
```
####  Deployment — Deep nesting example:

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3               # scalar
  selector:
    matchLabels:            # DICTIONARY
      app: my-app
  template:
    metadata:
      labels:               # DICTIONARY
        app: my-app
    spec:
      containers:           # LIST OF DICTIONARIES
        - name: my-app
          image: my-app:v1
          resources:        # DICTIONARY
            requests:       # DICTIONARY inside DICTIONARY
              cpu: "250m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          volumeMounts:     # LIST OF DICTIONARIES
            - name: config-vol
              mountPath: /etc/config
      volumes:              # LIST OF DICTIONARIES
        - name: config-vol
          configMap:        # DICTIONARY
            name: app-config
```