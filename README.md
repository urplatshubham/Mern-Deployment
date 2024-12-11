# README: MERN Application Deployment with Helm and Jenkins

This document provides step-by-step instructions for deploying a MERN-based application using Helm charts, automating the process with a Jenkins CI/CD pipeline, and detailing the steps performed for manual deployment using Kubernetes manifests.

The project consists of frontend and backend components, which are containerized and deployed to a Kubernetes cluster.

## **Prerequisites**

Before proceeding, ensure you have the following tools installed:

- **Docker**: For building container images.
- **Kubernetes (Minikube)**: For running the Kubernetes cluster.
- **Helm**: For managing Kubernetes deployments.
- **Jenkins**: For automating the build and deployment process.

---

## **1. Earlier Deployment Steps**

### **1.1 Backend Deployment**

1. **Build the Backend Docker Image:**

   ```bash
   docker build -t shubhamrajak2508/learner-backend:1.0 ./backend
   docker push shubhamrajak2508/learner-backend:1.0
   ```

2. **Create Kubernetes Deployment and Service Manifests:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: learner-backend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: learner-backend
     template:
       metadata:
         labels:
           app: learner-backend
       spec:
         containers:
         - name: backend
           image: shubhamrajak2508/learner-backend:1.0
           ports:
           - containerPort: 3000
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: backend-service
   spec:
       selector:
         app: learner-backend
       ports:
       - protocol: TCP
         port: 3000
         targetPort: 3000
       type: ClusterIP
   ```

3. **Apply the Backend Manifests:**

   ```bash
   kubectl apply -f backend-deployment.yaml
   kubectl apply -f backend-service.yaml
   ```

4. **Verify Deployment:**

   ```bash
   kubectl get pods
   kubectl get services
   ```

### **1.2 MongoDB Integration**

1. **Deploy MongoDB Using Kubernetes Manifests:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongo
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongo
     template:
       metadata:
         labels:
           app: mongo
       spec:
         containers:
         - name: mongo
           image: mongo:latest
           ports:
           - containerPort: 27017
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: mongo-service
   spec:
       selector:
         app: mongo
       ports:
       - protocol: TCP
         port: 27017
         targetPort: 27017
       type: ClusterIP
   ```

2. **Apply the MongoDB Manifests:**

   ```bash
   kubectl apply -f mongo-deployment.yaml
   ```

3. **Verify MongoDB Connection:**

   - Ensure the backend is connecting to MongoDB by checking the logs:
     ```bash
     kubectl logs <backend-pod-name>
     ```

### **1.3 Frontend Deployment**

1. **Build the Frontend Docker Image:**

   ```bash
   docker build -t shubhamrajak2508/learner-frontend:1.0 ./frontend
   docker push shubhamrajak2508/learner-frontend:1.0
   ```

2. **Create Kubernetes Deployment and Service Manifests:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: learner-frontend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: learner-frontend
     template:
       metadata:
         labels:
           app: learner-frontend
       spec:
         containers:
         - name: frontend
           image: shubhamrajak2508/learner-frontend:1.0
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: frontend-service
   spec:
       selector:
         app: learner-frontend
       ports:
       - protocol: TCP
         port: 80
         targetPort: 80
       type: NodePort
   ```

3. **Apply the Frontend Manifests:**

   ```bash
   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f frontend-service.yaml
   ```

4. **Access the Frontend Application:**

   ```bash
   minikube service frontend-service
   ```

---

## **2. Helm Chart Structure**

The Helm chart is organized as follows:

```
learner-chart/
├── Chart.yaml        # Chart metadata
├── values.yaml       # Default configuration values
└── templates/
    ├── deployment.yaml  # Deployment template
    ├── service.yaml     # Service template
```

### **2.1 Chart.yaml**

```yaml
apiVersion: v2
name: learner-chart
version: 0.1.0
```

### **2.2 values.yaml**

```yaml
replicaCount: 2

image:
  frontend: shubhamrajak2508/learner-frontend:1.0
  backend: shubhamrajak2508/learner-backend:1.0

service:
  type: NodePort
  port: 80

resources: {}
```

### **2.3 Deployment Template (templates/deployment.yaml)**

For both frontend and backend components:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: {{ .Values.image.frontend }}
        ports:
        - containerPort: {{ .Values.service.port }}
```

### **2.4 Service Template (templates/service.yaml)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}-service
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.port }}
  selector:
    app: {{ .Chart.Name }}
```

### **2.5 Deploy with Helm**

To deploy the application:

```bash
helm install learner-chart ./learner-chart
```

---

## **3. Jenkins CI/CD Pipeline**

The pipeline automates building Docker images, pushing them to Docker Hub, and deploying the application using Helm.

### **3.1 Jenkinsfile**

```groovy
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('docker-hub')
    }

    stages {
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_CREDENTIALS_USR/learner-frontend:1.0 ./frontend'
                sh 'docker build -t $DOCKER_CREDENTIALS_USR/learner-backend:1.0 ./backend'
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $DOCKER_CREDENTIALS_USR/learner-frontend:1.0'
                sh 'docker push $DOCKER_CREDENTIALS_USR/learner-backend:1.0'
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh 'helm upgrade --install learner-chart ./learner-chart'
            }
        }
    }
}
```

---

## **4. Testing and Validation**

1. **Verify Pods and Services**:

   ```bash
   kubectl get pods
   kubectl get services
   ```

2. **Access the Application**:

   - For NodePort services, use:
     ```bash
     minikube service <service-name>
     ```

3. **Check Logs**:

   ```bash
   kubectl logs <pod-name>
   ```

---

## **5. Repository Structure**

Ensure your GitHub repository is structured as follows:

```
.
├── frontend/          # Frontend application code
├── backend/           # Backend application code
├── learner-chart/     # Helm chart
├── Jenkinsfile        # Jenkins pipeline script
└── README.md          # Documentation
```

---

## **6. Prerequisites and Setup**

### **Docker Hub**

- Create a Docker Hub account.
- Store credentials in Jenkins as a secret (`docker-hub`).

### **Kubernetes Cluster**

- Use Minikube or any Kubernetes cluster.
- Install Helm on your local machine or Jenkins agent.

### **Jenkins Configuration**

- Install necessary plugins: Docker, Kubernetes CLI, and Pipeline.
- Add `docker-hub` credentials under **Manage Jenkins > Credentials**.
