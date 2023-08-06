
##### Overview on Helm
https://helm.sh/docs/intro/install/
https://artifacthub.io/

##### Download and Install Helm - From Apt (Debian/Ubuntu)

- Helm provides a single command-line client that is capable of performing all of the main Helm tasks
https://helm.sh/docs/intro/install/

``````sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

helm version
helm –h
``````
##### Helm commands cheatsheet

helm repo add	       Adds a Helm chart repository to the local cache list, after which we can reference it to pull charts from the repository
helm repo update	            Gets the latest information about chart repositories; the information is stored locally.
helm search repo	              Searches for charts in the given repositories.
helm pull	                     Downloads a given chart from the chart repository.
helm upgrade -i	             This will upgrade existing deployment of specific release:  helm upgrade <RELEASE NAME> <CHART NAME>
helm rollback                This will rollback a helm deployment to a specific revision number:  helm rollback <RELEASE NAME> <CHART NAME>
helm ls	                     Lists releases in the current namespace. If the -A flag is provided, it will list all the namespaces.
helm history	                 Prints historical revisions for a given release: helm history <RELEASE NAME>
helm template	                      Renders chart templates locally and displays the output.
helm create	         Creates a chart. This command will create the entire directory structure with all the files required to deploy e.g nginx
helm lint	                           Validates a chart
helm plugin	                         Installs, lists, updates, and uninstalls Helm plugins.
helm install                        helm install <RELEASE NAME> <CHART NAME>

##### 4.1 Adding a repo

The helm repo add command will add a repository named bitnami that points to the URL https://charts.bitnami.com/bitnami.
Once we have added a repository, its index will be locally cached until we next update it.

``````sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

``````
##### Searching a Chart repository

We can run a query inside the repo to look out for any specific chart.

``````sh
helm search repo drupal
helm search repo drupal --version
``````
##### Installing a Package (Chart)
At very minimum, installing a chart in Helm requires just two pieces of information:

- the name of the installation called release name - running chart
- the chart you want to install from repo

helm install <RELEASE NAME> <CHART NAME>
helm install chart-1 .    # with . as our current directory cd ~/charts/application-1
helm list -A    # list all releases

``````sh
# To install the drupal chart we will use:
helm install mysite bitnami/drupal

``````

##### Listing installed charts

``````sh
helm list

``````
##### 5. Create your first helm chart
- A Helm chart is an individual package that can be installed into your Kubernetes cluster.
- It is a collection of template files that describe Kubernetes resources.
- During chart development, you will often just work with a chart that is stored on your local filesystem.
- It uses templating to create Kubernetes manifests.

##### 5.1 Create a new chart
``````sh
 mkdir -p /k8s/helm-examples

helm create mychart  # This command will create the entire directory structure with all the files required to deploy e.g nginx
----
[root@controller helm-examples]# tree  mychart/

mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

``````
##### 5.2 Understanding helm chart’s structure

- Chart.yaml:
  The Chart.yaml file contains metadata and some functionality controls for the chart.
- charts:
   The folder where dependent sub-charts get stored.
- templates:
   Templates used to generate Kubernetes manifests are stored in the templates directory.
- NOTES.txt:
   This file is a special template. When a chart is installed, the NOTES.txt template is rendered and displayed rather than being installed into a cluster.
- tests: 
  Templates can include tests that are not installed as part of the install or upgrade commands. This chart includes a test that is used by the helm test command.
- values.yaml:
   Default values passed to the templates when Helm is rendering the manifests are in the values.yaml file. When you instantiate a chart, these values can be overridden.

##### 5.3 Modifying the chart’s values

We will make some minor modifications to the default chart templates and values.
Under mychart/values.yaml we will modify pullPolicy to Always. The default value i.e. IfNotPresent means the pod will avoid pulling an image if it already exists.
This we change to Always to pull the image every time a pod is deployed.

``````sh
# ex. Namespace: to read the values keyword
Kind: {{ .Values.key }}
Kind: {{ .Values.type }}
lables:
  name: Kind: {{ .Values.labels.name }}

---To
#values.yaml

type: Namespace
labels:
  name: demo-a

``````
- Under mychart/values.yaml we will use NodePort service to access the nginx server instead of ClusterIP from external network.

``````sh
--From
service:
  type: ClusterIP
  port: 80

---To
service:
  type: NodePort
  port: 80

``````

##### 5.5 Linting the helm chart
- We can utilise helm's linting feature to check the chart for possible issues and errors
- check the Helm chart content by running a series of tests to verify the chart integrity.
we can use the helm lint <CHART NAME> command
``````sh
helm lint mychart

helm list
kubectl get nodes
kubectl get ns

``````
##### 5.6 Installing the helm chart
- It is always recommended to test your application before install and upgrade using --dry-run with helm command.
- With this, Helm will validate the templates against the Kubernetes cluster.
- the --dry-run was successfully able to trigger the deploy hence it is safe to install the chart with status: pending-install
- Next we will actually install the chart without using --dry-run. We will name our chart as nginx: and check the status: deployed

helm install <RELEASE NAME> <CHART NAME>
``````sh
helm install nginx mychart/ --dry-run
--
heml install chart1 . --dry-run

``````
##### Helm Upgrade Install
whenever we make changes to our chart and want to deploy
``````sh
helm upgrade <name of chart> .
or 
helm upgrade --install <name of chart> . --dry-run

helm list
helm history chart_name

``````
##### 5.7 List all the resources deployed by chart
- Now we can check the --dry-run output to get the list of resources which will be deployed by our chart,
-  It expects the Release Name of the chart

``````sh
kubectl get all -l "app.kubernetes.io/name=mychart"

``````
##### 5.8 Accessing the nginx server
we had defined a Service with NodePort so we can use the same to access the nginx server using external network.

``````sh
kubectl get svc
kubectl get pods -owide

``````
##### 6. Debugging Helm Chart Templates
It can be really tricky to debug a helm chart template to identify the cause of failure.
ome of the commands which can come handy in debugging helm chart templates:

- helm lint:   is your go-to tool for verifying that your chart follows best practices
- helm install --dry-run --debug or helm template --debug:    We've seen this trick already. It's a great way to have the server render your templates, then return the resulting manifest file.
- helm get manifest:     This is a good way to see what templates are installed on the server.

``````sh
helm get manifest nginx

``````
##### 6. Deleting/Un-installing a chart

``````sh
helm uninstall chart_name
``````
##### 6. Conditional flow control - if/else block
Here the condition will be evaluated by the Pipeline to see whether the condition is true or false. The basic structure of if/else block is this:

``````sh
{{ if VALUE or PIPELINE }}  
# Do something
{{ else if OTHER VALUE or OTHER PIPELINE }}  
# Do something else
{{ else }}   
# Default
{{- end }}
``````
Here are our values.yaml and service.yaml:

``````sh
replicaCount: 1
image:
  repository: nginx
  tag: 1.21.6
  pullPolicy: Always
env: uat

---
replicaCount: 1
image:
  repository: nginx
  tag: 1.21.6
  pullPolicy: Always
env: sit
``````
templates/service.yaml:
if in our values.yaml file environment variable will be there then the template will generate with the label env otherwise it will not
``````sh
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  {{- if .Values.env }}
  labels:
    env: {{ .Values.env }}
  {{- end }}
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
``````
The actual service.yaml file will be:
``````sh
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  labels:
    env: uat
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
``````
##### Note:
We add “-” sign next to the curly brackets to avoid extra whitespace/newlines in output.

``````sh
This means remove whitespaces/newlines from the left.
{{-

This means remove whitespaces/newlines from the right.
-}}
``````
Now Suppose in our case if environment is “uat” or any other like cug/prod, we need to print env label as it is
templates/service.yaml
``````sh
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  {{- if .Values.environment }}
  labels:
    env: {{ .Values.environment }}
  {{- else if eq .Values.environment "sit" }}
  labels:
    env: dev
  {{- end }}
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
``````
There are some other expressions in Helm that we can use in these types of situations:
``````sh
eq ---> equal
ne ---> not equal
lt ---> less than
le ---> less than or equal to
gt ---> greater than
ge ---> greater than or equal to
not ---> negation
empty ---> value is empty
``````
##### 6. with
with helps us to modify the scope. In the template, the current scope is “.”
Here is the structure of with -
``````sh
{{ with PIPELINE }}
  # restricted scope
{{ end }}

``````
values.yaml file
``````sh
tags:
  kafka:
    uname: myKafka
    topic: test
  database:
    uname: mongo
    collection: users
``````
These data we need to pass in our configmap.yaml. So our configmap.yaml will be
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: {{ .Release.Name }}
data:
   kafka-username: {{ .Values.tags.kafka.uname }}
   kafka-topic: {{ .Values.tags.kafka.topic }}
   database-username: {{ .Values.tags.database.uname }} 
   database-collection: {{ .Values.tags.database.collection }}
``````
the duplication of .Values.tags, we’ve written this 4 times.
And we can avoid this by setting up the scope using with block.

change the scope and make our configmap.yaml more simple. The scope is .Values.tags
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: {{ .Release.Name }}   # this release.name will throw an error as it is within the scope
data:
   {{- with .Values.tags }}
   kafka-username: {{ .kafka.uname }}
   kafka-topic: {{ .kafka.topic }}
   database-username: {{ .database.uname }} 
   database-collection: {{ .database.collection }}
   {{- end }}

``````
But we can make it more simple by removing the duplication of kafka and database with the help of with block.

``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: {{ .Release.Name }}
data:
   {{- with .Values.tags }}
    {{- with .kafka }}
     kafka-username: {{ .uname }}
     kafka-topic: {{ .topic }}
    {{- end }}
    {{- with .database }
     database-username: {{ .uname }} 
     database-collection: {{ .collection }}
    {{- end }}
   {{- end }}

``````
If we just put it like this it will throw an error names nil pointer evaluating because we are currently in the scope .Values.tags and there is no Release object.
Here Release object is in the root scope. Root scope is also represented by the $ sign. So we can use $ here before the . and it will take us to root scope.
``````sh
release: {{ .Release.Name }}

release: {{ $.Release.Name }}
``````
##### 7. Range
range is as similar to for/foreach loops in any other programming language. We can use range to iterate through a collection one by one.

values.yaml file
``````sh
secrets:
  - mongo-secret
  - kafka-secret
  - redis-secret
  - vault-secret

``````
Now if we need to pass this data, our configmap.yaml looks like this:

``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: {{ .Release.Name }}
data:
   secrets:
     - mongo-secret
     - kafka-secret
     - redis-secret
     - vault-secret
``````
Let’s see how our configmap.yaml will look like using range:
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: {{ .Release.Name }}
data:
   secrets:
   {{- range .Values.secrets }}
     - {{ . }}
   {{- end }}
``````
##### Inbuilt Object
Release: This object describes the release itself. It has several objects inside of it:
``````sh
Release.Name: The release name
Release.Namespace: The namespace to be released into (if the manifest doesn’t override)

``````
###### Indent spaces
The indent of 4 or 8 is dependent on the current indenting going on with the YAML file. If you need the output to be 4 spaces then you use 4, if you need the output to be 8 space (to align with the surrounding YAML, then you use 8)
``````sh
apiVersion: v1
kind: Service
metadata:
  name: {{ include "anvil.fullname" . }}
  labels:
    {{- include "anvil.labels" . | nindent 4 }}
spec:

``````
So in this example, the output is being indented 4 spaces. When the output is generated via helm template anvil you'll see this:
Notice that the output from the template has 4 spaces before each line, so the output is clearly part of the labels: property.
``````sh
# Source: anvil/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: RELEASE-NAME-anvil
  labels:
    helm.sh/chart: anvil-0.1.0               # <<<--- From Template
    app.kubernetes.io/name: anvil            # <<<--- From Template
spec:
``````
If we changed the indent to 8 the syntax would look like this:
``````sh
# Source: anvil/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: RELEASE-NAME-anvil
  labels:
        helm.sh/chart: anvil-0.1.0               # <<<--- From Template
        app.kubernetes.io/name: anvil            # <<<--- From Template

spec:
``````
##### Include statement
 include is a template-processing directive which tells helm to reference a template function defined elsewhere (usually _helpers.tpl).
``````sh
{{ include "toYaml" $value | indent 2 }}
labels: {{- include "mylabels" . | nindent 4 }}

``````
##### _helpers.tpl
A file named _helpers.tpl is usually used to define Go template helpers with this syntax:
Th helpers is generally to reduce multiple labels, or objects used in a template.
nindent: new line indentation. 
``````sh
{{- define "mylabels" -}}
app: nginx
location: frontend
server: proxy
{{- end -}}

---
{{- $v := include "sometpl" . | fromYaml }}
some: {{- $v | toYaml | nindent 2 }}
``````
``````sh
labels: {{- include "mylabels" . | nindent 4 }}
``````
``````sh

``````