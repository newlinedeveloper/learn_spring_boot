### **Phase 5: Deployment & Monitoring 🚀**  

Now, let's cover **deploying & monitoring** Spring Boot applications, including **Docker, Kubernetes, Helm, CI/CD, Prometheus, and Grafana**.

---

## **1. Dockerizing Spring Boot Applications**  

### **Step 1: Create a Dockerfile**
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### **Step 2: Build & Run the Docker Image**
```bash
# Build the Docker image
docker build -t myapp .

# Run the container
docker run -p 8080:8080 myapp
```

✅ Your Spring Boot app is now running inside a **Docker container**.

---

## **2. Kubernetes Deployment with Helm Charts**  

### **Step 1: Create a Helm Chart**
```bash
helm create myapp
```

### **Step 2: Define Deployment (`templates/deployment.yaml`)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 8080
```

### **Step 3: Deploy to Kubernetes**
```bash
helm install myapp ./myapp
```

✅ Now, your app is running on **Kubernetes with Helm**.

---

## **3. CI/CD Pipeline using GitHub Actions**  

### **Step 1: Create `.github/workflows/deploy.yml`**
```yaml
name: Deploy to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build JAR
        run: mvn clean package -DskipTests

      - name: Build Docker Image
        run: docker build -t myapp:latest .

      - name: Push to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker tag myapp:latest mydockerhub/myapp:latest
          docker push mydockerhub/myapp:latest

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
```

✅ Now, **GitHub Actions will automate your CI/CD pipeline**.

---

## **4. Monitoring with Prometheus & Grafana**  

### **Step 1: Add Prometheus Dependency**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### **Step 2: Enable Prometheus Metrics**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

### **Step 3: Deploy Prometheus & Grafana**
```bash
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```

✅ Now, **Grafana dashboards will show real-time metrics**.

---
