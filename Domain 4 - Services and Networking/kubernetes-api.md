#### Interacting directly with the API

Verify the currently available Kubernetes API versions on the cluster:

``````sh
kubectl --help

kubectl api-versions

kubectl proxy --port=8001

# Open up another terminal by clicking the + button and select Open New Terminal.

curl localhost:8001 or curl -X GET http://localhost:8001

# You can explore the OpenAPI definition file to see complete API details

curl localhost:8001/apis/apps/v1

# on a browser:

http://127.0.0.1:8081/apis/apps/v1

``````