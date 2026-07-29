# Creating a Custom Helm Chart for AI-BankApp

## Task 1: Scaffold the Chart and Study the Raw Manifests

Map each file to what it does:

| File | Purpose |
|------|---------|
| `configmap.yml` | MySQL host, port, database, Ollama URL |
| `secrets.yml` | MySQL credentials (base64 encoded) |
| `bankapp-deployment.yml` | BankApp with init containers, probes, envFrom |
| `mysql-deployment.yml` | MySQL with EBS volume mount, probes |
| `ollama-deployment.yml` | Ollama with postStart model pull, probes |
| `service.yml` | ClusterIP services for all 3 components |

Now scaffold a Helm chart:
```bash
mkdir helm-chart && cd helm-chart
helm create bankapp
```

Delete the generated template files -- you will write your own from the raw manifests:
```bash
rm -rf bankapp/templates/*.yaml bankapp/templates/tests/
```

Keep `_helpers.tpl` and `NOTES.txt` -- you will customize them.

---




## Task 2: Define Chart.yaml and values.yaml

**chart.yml**
```
apiVersion: v2
name: bankapp
description: A SpringBoot bankapp application with mysql DB and with AI ChatBot using Ollama.
type: application
version: 0.1.0
appVersion: "1.0.0"
maintainers:
  - name: RohitKumar
    url: https://github.com/rohit5126/AI-BankApp-DevOps.git
keywords:
  - bankapp
  - spring-boot
  - mysql
  - ollama
  - ai

```

**values.yml**
```
# Bankapp Configuration
bankapp:
  replicaCount: 2
  image: 
    repository: rohit5126/bankapp
    tags: "latest"
    pullPolicy: Always
  resources:  
    requests:
      memory: "256Mi"
      cpu: "256m"
    limits:
      memory: "512Mi"
      cpu: "512m"
  service:
    type: NodePort
    port: 8080
    nodePort: 30080
  autoscaling:
    enabled: true
    minReplicas: 2
    maxreplica: 4
    targetCPUUtilization: 70

#mysql stateful

mysql:
  replicaCount: 1
  image:  
    repository: mysql
    tags: "8.0"
  service:
    type: ClusterIP
    port: 3306
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard

  

#ollama deployment

ollama:
  replicaCount: 1
  image:
    repository: ollama/ollama
    tags: "latest"
  resources:
    requests:
      memory: "1Gi"
      cpu: "500m"
    limits:
      memory: "2Gi"
      cpu: "1000m"
  service:
    type: ClusterIP
    port: 11434

#secrets 

secret: 
  MYSQL_ROOT_PASSWORD: bankapp
  MYSQL_DATABASE: bankapp

#configMap

config:
  MYSQL_HOST: mysql-state-0.mysql.default.svc.cluster.local

```

## Task 3: Write the Core Templates

**configmap.yml**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-host
data:
  MYSQL_HOST: {{ .Values.config.MYSQL_HOST | quote }}
```

**secret.yml**
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: {{ .Values.secret.MYSQL_ROOT_PASSWORD | quote }}
  MYSQL_DATABASE: {{ .Values.secret.MYSQL_DATABASE | quote }}
```

**statefulset.yml**

```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: {{ .Values.mysql.service.type }}
  clusterIP: None
  selector:  
    app: mysql-app
  ports:
    - port: {{ .Values.mysql.service.port }}
      targetPort: {{ .Values.mysql.service.port }}

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-state
  labels: 
    app: mysql-app
spec:
  selector:
    matchLabels:
      app: mysql-app
  serviceName: mysql
  replicas: {{ .Values.mysql.replicaCount }}
  template:
    metadata:
      labels:
        app: mysql-app
    spec:
      containers:
        - name: mysql-state
          image: "{{ .Values.mysql.image.repository }}:{{ .Values.mysql.image.tags }}"
          ports:
            - containerPort: {{ .Values.mysql.service.port}}
              name: web
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
          env:
            - name: MYSQL_PASSWORD
              valueFrom: 
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            
            - name: MYSQL_DATABASE
              valueFrom: 
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE
          readinessProbe:
            exec:
              command: ["sh","-c", "mysqladmin ping -h localhost -u root -p$MYSQL_PASSWORD"]
            initialDelaySeconds: 30
            periodSeconds: 10
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        storageClassName: {{ .Values.mysql.storageClassName | quote }}
        accessModes: [ "ReadWriteOnce" ]
        {{- with .Values.mysql.resources }}
        resources:
            {{- toYaml . | nindent 12 }}
        {{- end }}

```

**deployment.yml**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama-deployment
spec:
  replicas: {{ .Values.ollama.replicaCount }}
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      containers:
      - name: ollama-app
        image: "{{ .Values.ollama.image.repository }}:{{ .Values.ollama.image.tags }}"
        command:
          - /bin/bash
          - -c
          - |
            echo "renaming file from .so to .bak"
            if [ -f /usr/lib/ollama/libggml-cpu-sapphirerapids.so ]; then
              mv /usr/lib/ollama/libggml-cpu-sapphirerapids.so /usr/lib/ollama/libggml-cpu-sapphirerapids.so.bak
            fi

            echo "Starting Ollama server in background..."
            /bin/ollama serve &
            SERVER_PID=$!

            echo "Waiting for Ollama server to become available..."
            while ! (: </dev/tcp/127.0.0.1/11434) 2>/dev/null; do
              sleep 1
            done

            echo "Pulling tinyllama model..."
            /bin/ollama pull tinyllama

            echo "Ollama is ready!"
            wait $SERVER_PID
        env:
          - name: OLLAMA_CPU_AVX2
            value: "false"
        {{- with .Values.ollama.resources }}
        resources:
            {{- toYaml . | nindent 12 }}
        {{- end }}
        ports:
          - containerPort: {{ .Values.ollama.service.port }}
        volumeMounts:
          - name: ollama-data
            mountPath: "/root/.ollama"
        readinessProbe:
            exec:
              command: ["sh","-c", "ollama list"]
            initialDelaySeconds: 30
            periodSeconds: 50
            timeoutSeconds: 50
      volumes:
        - name: ollama-data
          emptyDir: {}

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: bankapp-dep
  labels: 
    app: bankapp
spec:
  replicas: {{ .Values.bankapp.replicaCount }}
  selector:
    matchLabels:
      app: bankapp
  template:
    metadata:
      labels:
        app: bankapp
    spec:
      containers:
        - name: bankapp-container
          image: "{{ .Values.bankapp.image.repository }}:{{ .Values.bankapp.image.tags }}"
          ports:
            - containerPort: {{ .Values.bankapp.service.port }}
          {{- with .Values.bankapp.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          env: 
            - name: MYSQL_PASSWORD
              valueFrom: 
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            
            - name: MYSQL_DATABASE
              valueFrom: 
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE
            - name: MYSQL_HOST
              valueFrom:  
                configMapKeyRef:
                  name: mysql-host
                  key: MYSQL_HOST
            - name: MYSQL_PORT
              value: {{ .Values.mysql.service.port | quote }}
            - name: MYSQL_USER
              value: root

            - name: OLLAMA_URL
              value: http://ollama:11434

```

**service.yml**
```
apiVersion: v1
kind: Service
metadata:
  name: ollama
spec:
  type: {{ .Values.ollama.service.type }}
  selector:
    app: ollama
  ports:
  - port: {{ .Values.ollama.service.port }}
    targetPort: {{ .Values.ollama.service.port }}

---

apiVersion: v1
kind: Service
metadata:
  name: bankapp-svc
spec:
  selector:
    app: bankapp
  type: {{ .Values.bankapp.service.type }}
  ports:
  - port: {{ .Values.bankapp.service.port }}
    nodePort: {{ .Values.bankapp.service.nodePort }}
    targetPort: {{ .Values.bankapp.service.port }}

```

## Task 6: Validate and Deploy

**Lint the chart:**

`helm lint bankapp/`

**Render templates locally -- see the final YAML without deploying:**

`helm template my-bankapp bankapp/`

Review the output. Every {{ }} should be resolved to actual values.

**we can also run by overriding the values**
```
helm install my-bankapp bankapp/ \
> --set bankapp.replicaCount=1 \
> --set ollama.enabled=false
```


