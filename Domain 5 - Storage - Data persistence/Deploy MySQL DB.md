
https://dev.to/musolemasu/deploy-a-mysql-database-server-in-kubernetes-static-dpc

##### Deploy a MySQL database server in Kubernetes - Static Provisioning

1- Build a Persistent Volume (PV)
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
EOF
``````
2-  Build a Persistent Volume Claim (PVC)
``````sh
cat << EOF| kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
``````
3- Build the Service  
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  clusterIP: None
EOF
``````
3- Create a secret.yml file for MySQL that will be mapped as an Environment Variable as follows:
``````sh
echo -n 'password1' | base64
cGFzc3dvcmQx

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  password: cGFzc3dvcmQx
EOF
``````
3- Deploy the MySQL BD
``````sh
cat<< EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql:8.0
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password   # the one generated before in secret.yml
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
EOF
``````
###### Create and Test the objects
``````sh
kubectl apply -f mysql-pv.yaml
kubectl apply -f mysql-pvc.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
----
kubectl get deployments
kubectl get pods
kubectl get pv,pvc
kubectl describe pods pod-name
kubectl get services

``````
###### Testing the Mysql Deployment/Access the MySQL Database inside the cluster
Now that the MySQL is deployed, we would like to connect to the node right inside the cluster
now run a test to create a Pod running a MySQL container that connects to the MySQL database server Pod as a client
``````sh
kubectl exec -it po/mysql-66cbddd4fc-nlj86 -- bash
mysql -h mysql -u root -p 
SHOW DATABASES;
exit
-- # test the services

kubectl run testpod1 --image=ubuntu -- sh -c 'sleep 3600'
kubectl exec -it po/testpod1 -- sh
--
apt update && upgrade
apt install dnsutils -y
nslookup mysql
--
kubectl scale deploy/mysql --replicas=4
nslookup mysql
exit

``````