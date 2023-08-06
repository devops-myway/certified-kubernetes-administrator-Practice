https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

https://amperecomputing.com/tutorials/deploy-wordpress-and-mysql
https://kubesphere.io/docs/v3.3/quick-start/wordpress-deployment/
https://awstip.com/lightning-fast-wordpress-and-mysql-deployment-on-aws-eks-with-ebs-using-k8s-manifests-1bef0aef355b
https://medium.com/@containerum/how-to-deploy-wordpress-and-mysql-on-kubernetes-bda9a3fdd2d5


####  Create the Deployment Yaml for MySQL - Backend
This deployment file would have three parts to Four parts:
Service - Headless Service, Secret, PersistentVolumeClaim and Deployment.

###### Build a Persistent Volume (PV) - MySql
The MySQL container uses a PersistentVolume (PV) for storage using the host path /tmp/mysql/data and mounted at /var/lib/mysql
``````sh
$ mkdir mysql-kube
$ cd mysql-kube/
------
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/mysql/data"
  persistentVolumeReclaimPolicy: Recycle
EOF

``````
###### Build a Persistent Volume Claim (PVC) - Mysql
A PersistentVolumeClaim(PVC) is a request for storage (PV) by a user. Think of it as a pod made specially for providing storage space for other pods.
``````sh
cat << EOF | kubectl apply -f -
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
      storage: 20Gi
EOF

``````
##### Service Headless - which in the below case is not tested for the clusterIP service
``````sh
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress-mysql
spec:
  clusterIP: None
  ports:
  - port: 3306
  selector:
    app: wordpress-mysql
``````
###### Deployment for Mysql-DB
``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress-mysql
spec:
  selector:
    matchLabels:
      app: wordpress-mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress-mysql
    spec:
      containers:
      - image: mysql:oracle
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password.txt  # The password for the MySQL database is stored in a Kubernetes secret which is generated from password.txt.
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
``````
##### PREPARING THE DEPLOYMENT FILE FOR WORDPRESS - Frontend
The WordPress container also uses a PersistentVolume for storing the website files using the host path /tmp/wp/data and mounted at /var/www/html
The environment variable WORDPRESS_DB_HOST sets the name of the MySQL service and WORDPRESS_DB_PASSWORD defines the password to access the MySQL database from the Kubernetes secret generated from the password.txt file.

If using a cloud deployment, you can set the service type to LoadBalancer and use the IP address of the cloud load balancing service to access the WordPress front-end.
If using Kubernetes on bare metal, you can set the type to NodePort and use the Kubernetes node IP address to access WordPress front-end.

##### Deploy the PV for WordPress
``````sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wp-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/wp/data"
  persistentVolumeReclaimPolicy: Recycle

``````
##### Deploy the PVC for Wordpress
``````sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

``````
##### Deploy the Service for Wordpress
``````sh
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
  externalIPs:
  - 10.76.87.233
``````
##### Deployment of the Wordpress 
``````sh
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wp-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/wp/data"
  persistentVolumeReclaimPolicy: Recycle
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
  externalIPs:
  - 10.76.87.233
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:6.0.1-php7.4
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password.txt
          ports:
            - containerPort: 80
              name: wordpress
          volumeMounts:
            - name: wordpress-persistent-storage
              mountPath: /var/www/html
      volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wp-pv-claim
``````
###### INSTALLATION STEPS
Set up MySQL secret. Create a new file called password.txt and add your desired MySQL password in this file.
``````sh
kubectl create secret generic mysql-pass --from-file=password.txt
kubectl create -f mysql-deployment.yaml
kubectl create -f wordpress-deployment.yaml

``````
###### VERIFY SERVICES AND INSTALL WORDPRESS
Verify that services are running using the following command:
``````sh
kubectl get service
kubectl get service
``````

Copy the external IP address of the WordPress service and open the link http://<"ip-address"> in a browser.
Fill in the details and set up the WordPress site.