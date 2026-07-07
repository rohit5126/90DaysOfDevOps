# Kubernetes ConfigMaps and Secrets

`ConfigMaps and Secrets are core Kubernetes API resources used to decouple configuration data from your application container images. 
configMaps Stores non-sensitive configuration parameters. secrets Storing sensitive, confidential information.`

## Task 1: Create a ConfigMap from Literals

```

kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_DEBUG=false
kubectl describe configmaps app-config 
kubectl get configmaps app-config -o yaml

#the data stored in a normal text form
```

## Task 2: Create a ConfigMap from a File

```
  307  kubectl create configmap nginx-config --from-file=default.conf=nginx.conf 
  308  kubectl describe configmaps nginx-config

The key name (default.conf) becomes the filename when mounted into a Pod
```

## Task 3: Use ConfigMaps in a Pod


you can use any file and create a config map with it using 
kubectl create configmap nginx-config --from-file=default.conf=<file_name>

this will create a configmap with name nginx-config with data default.conf, whihc will contain all the text or scrpt from the file.
you can use this config inside volumes while creating deployment and add it to conatiner inside volumemounts with a mount path, which will
create a file with name default.conf or all the data inside the configmap will create a file in the path with name as data-name.

```
nginx.conf

#server {
        listen       80;
        server_name  localhost;

        location /health {
            access_log off;
            add_header Content-Type text/plain;
            return 200 'healthy server everything working fine';
        }

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }

configmap
kubectl create configmap nginx-config --from-file=default.conf=nginx.conf

pod
##
apiVersion: v1
kind: pod
metadata:
  labels:
  apps: my-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: nginx-con 
          mountPath: "/etc/nginx/conf.d" 
          readOnly: true
  volumes:
    - name: nginx-con
      configMap:
        name: nginx-config 
```
to verify-

kubectl exec <pod_name> -- curl -s http://localhost/health

## Task 4: Create a Secret





