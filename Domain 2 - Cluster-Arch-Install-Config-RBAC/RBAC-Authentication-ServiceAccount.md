##### ServiceAccount Documentation link:

https://kubernetes.io/docs/reference/access-authn-authz/authentication/

-------Notes:
Service accounts are usually created automatically by the API server and associated with pods running in the cluster through the Serviceccount. The default ServiceAccount that lives in the default namespace. Any Pod that doesn’t explicitly assign a ServiceAccount uses the default ServiceAccount.

Kubernetes uses a ServiceAccount to authenticate the Helm service or some service applications running inside of a pod, with the API  authentication server through a token. This ServiceAccount can be assigned to a Pod and mapped to RBAC rules.

#### Step 1: Create a new Service Account and assign a role: imaparatively
```sh
kubectl create serviceaccount demo-svc

# List the serviceaccount

kubectl get serviceaccounts

# Render serviceaccount details and Find the name of the secret that stores the service account’s token

kubectl describe serviceaccount demo-svc

# Identify the secret

kubectl get secrets

# Retrieve the token’s value from the secret using the following command:

TOKEN=$(kubectl describe secret demo-token-znwmb | grep token: | awk '{print $2}')

echo $TOKEN

```
#### Step 2: With the token extracted, you can add your user as a new kubectl context:
```sh

kubectl config set-credentials demo --token=$TOKEN

kubectl config set-context demo --cluster=kubernetes --user=demo

kubectl config current-context

kubectl config use-context demo

# Try to retrieve the pods in your cluster: see the error
kubectl get pods

```

#### Step 3: Creating a Role

Switch back to your original context before continuing, so you regain your administrative privileges:

To keep things simple, you’ll use the “Developer” role example from earlier:

```sh

kubectl config use-context default

---------------------------------------
nano role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: Developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "get", "list"]

----------------------------------------
kubectl apply -f role.yaml

```
#### Step 4: Creating a RoleBinding
A role on its own has no effect on your cluster’s access control. It needs to be bound to users with a RoleBinding.
Create this object now to grant your demo service account the permissions associated with your Developer role.

```sh
nano role-binding.yaml
-----------------------------------------
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: DeveloperRoleBinding
subjects:
  - kind: ServiceAccount
    name: demo
    apiGroup: ""
roleRef:
  kind: Role
  name: Developer
  apiGroup: ""
---------------------------------------
kubectl apply -f role-binding.yaml

```

#### Step 5: Testing Your RBAC Policy
```sh
kubectl config use-context demo

kubectl get pods

kubectl delete pods

```
