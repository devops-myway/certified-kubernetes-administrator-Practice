#### Interacting directly with the API

Verify the currently available Kubernetes API versions on the cluster:
using the curl is a command-line tool to transfer data to or from a server, using any of the supported protocols e.g http, https 
curl [options] [URL...]
curl https://www.geeksforgeeks.org

``````sh
kubectl --help

kubectl api-versions

kubectl proxy --port=8001

# Open up another terminal by clicking the + button and select Open New Terminal.

curl localhost:8001
curl localhost:8001/apis/v1
curl localhost:8001/apis/apps/v1

# on a browser:

http://127.0.0.1:8081/apis/apps/v1

``````