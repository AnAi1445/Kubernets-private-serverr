1. Cluster Setup (Private Networking)
Assume the cluster is already initialized with kubeadm or a managed cloud provider. Ensure private networking is enabled.

2. Create Namespace
Create a namespace to isolate your DEX environment:
...
apiVersion: v1
kind: Namespace
metadata:
  name: dex-swap-bridge

...
Apply it:

...

kubectl apply -f namespace.yaml
...

3. Deploy Backend Services
Deployment Configuration (dex-backend-deployment.yaml):

...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-backend
  namespace: dex-swap-bridge
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dex-backend
  template:
    metadata:
      labels:
        app: dex-backend
    spec:
      containers:,
      - name: backend
        image: my-dex-backend-image:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          value: "postgresql://dex_user:password@dex-database:5432/dex_db"

,....
Service Configuration (dex-backend-service.yaml):
...
4. Deploy Frontend Services
Deployment Configuration (dex-frontend-deployment.yaml):

...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-frontend
  namespace: dex-swap-bridge
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dex-frontend
  template:
    metadata:
      labels:
        app: dex-frontend
    spec:
      containers:
      - name: frontend
        image: my-dex-frontend-image:latest
        ports:
        - containerPort: 80

...

Service Configuration (dex-frontend-service.yaml):

...

apiVersion: v1
kind: Service
metadata:
  name: dex-frontend
  namespace: dex-swap-bridge
spec:
  type: LoadBalancer
  selector:
    app: dex-frontend
  ports:
  - port: 80
    targetPort: 80



...

Deploy the frontend
....

kubectl apply -f dex-frontend-deployment.yaml
kubectl apply -f dex-frontend-service.yaml
....
5. Deploy Bridge Infrastructure
Deployment Configuration (dex-bridge-deployment.yaml):


...,

apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-bridge
  namespace: dex-swap-bridge
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dex-bridge
  template:
    metadata:
      labels:
        app: dex-bridge
    spec:
      containers:
      - name: bridge
        image: my-dex-bridge-image:latest
        ports:
        - containerPort: 9090
        env:
        - name: PRIVATE_KEY
          valueFrom:
            secretKeyRef:
              name: dex-secrets
              key: private-key


...
Service Configuration (dex-bridge-service.yaml):
...



apiVersion: v1
kind: Service
metadata:
  name: dex-bridge
  namespace: dex-swap-bridge
spec:
  type: ClusterIP
  selector:
    app: dex-bridge
  ports:
  - port: 9090
    targetPort: 9090
...
Deploy the bridge
...

apiVersion: v1
kind: Service
metadata:
  name: dex-bridge
  namespace: dex-swap-bridge
spec:
  type: ClusterIP
  selector:
    app: dex-bridge
  ports:
  - port: 9090
    targetPort: 9090
...
Deploy the bridge
...


kubectl apply -f dex-bridge-deployment.yaml
kubectl apply -f dex-bridge-service.yaml
...
Generate Secrect

Generate a private key for the bridge and store it securely:
...

kubectl create secret generic dex-secrets \
  --namespace dex-swap-bridge \ 
  --from-literal=private-key=$(openssl rand -hex 32)

...

7. Testing the Setup
Use kubectl to ensure all pods are running:



...
kubectl get pods -n dex-swap-bridge
...

Expose the frontend via the LoadBalancer IP. Test it by visiting:

Expose the frontend via the LoadBalancer IP. Test it by visiting:
,,,,


8. Random ID and Password Script
Run this Bash script to generate IDs for different components:




...#!/bin/bash

generate_random_credentials() {
  RANDOM_ID=$(openssl rand -hex 12)
  RANDOM_PASSWORD=$(openssl rand -base64 16)
  echo "Component: $1"
  echo "Random ID: $RANDOM_ID"
  echo "Random Password: $RANDOM_PASSWORD"
}

echo "Generating Random Credentials for DEX Components:"
generate_random_credentials "Backend"
generate_random_credentials "Frontend"
generate_random_credentials "Bridge"

,,,,


Output
...

 noGenerating Random Credentials for DEX Components:
Component: Backend
Random ID: 8f3b4e5a6d7c
Random Password: hKv9P+Bbq8jLtx==

Component: Frontend
Random ID: 4d3e7f8g6h9k
Random Password: R3wmXp1tKl==

Component: Bridge
Random ID: 7c8d6b5a9e4f
Random Password: L0jKp2tVy==z

....


