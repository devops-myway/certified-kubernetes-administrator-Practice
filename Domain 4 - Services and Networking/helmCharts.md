
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

helm repo add	             Adds a Helm chart repository to the local cache list, after which we can reference it to pull charts from the repository
helm repo update	            Gets the latest information about chart repositories; the information is stored locally.
helm search repo	              Searches for charts in the given repositories.
helm pull	                     Downloads a given chart from the chart repository.
helm upgrade -i	             This will upgrade existing deployment of specific release:  helm upgrade <RELEASE NAME> <CHART NAME>
helm rollback                This will rollback a helm deployment to a specific revision number:  helm rollback <RELEASE NAME> <CHART NAME>
helm ls	                     Lists releases in the current namespace. If the -A flag is provided, it will list all the namespaces.
helm history	                 Prints historical revisions for a given release: helm history <RELEASE NAME>
helm template	                      Renders chart templates locally and displays the output.
helm create	                  Creates a chart. This command will create the entire directory structure with all the files required to deploy e.g nginx
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
helm install --dry-run nginx mychart/
--
heml install nginx mychart/

``````
##### Helm Upgrade Install
whenever we make changes to our chart and want to deploy
``````sh
helm upgrade <name of chart> .
or 
helm upgrade --install <name of chart> .

helm list

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

# Check the worker node on which our deployment pod is running on:
kubectl get pods -owide

# So we can use the worker-1.example IP with 31204 port from PORT(S) section of kubernetes service output to access the nginx server from nginx-mychart-7fd98b7fd-mmx62:

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

helm uninstall mysite
``````
##### 6. Using - With Statement for rendering repitition

``````sh

{{ - with .Values.key }}

template for labels, spec etc

{{- end }}
``````
##### 6. Reduce code repition using $ statement

``````sh
values.yaml

service:
  type:Nodeport
  labels:
    app: nginx
  port:80
  targetport:8080
  
# not under services and we use this $ sign to fix the repitition
replicacount: 2
imagenginx: nginx:1.16
-----
deployment.yaml

{{ - $replica := .Values.replicacount }}
{{ - $image := .Values.imagenginx }}
{{ - with .Values.service }}

spec.templ;ate.spec.conatiners

replicas: {{ $replicas }}
image: {{ $image }}

``````
##### 7. Range with $key $ value to iterate all over the values

``````sh
values.yaml

config:
  name: postgres-config
  data:
  - key: POSTGRES_DB
    value: postgress
  - key: POSTGRES_USER
    value: shan
  - key: POSTGRES_PASSWORD
    value: secretpass
  

-----
postgres-config.yaml

metadata:
{{ .Values.postgres.config.name }}
labels:
  group: {{ .Values.postgres.group }}

data:
{{ - range .Values.postgres.config.data }}
  {{ .key}}: {{ .value }}
{{ - end }}

``````