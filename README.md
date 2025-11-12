1. Git Essentials – Commit, Push, Merge, and Pull Requests
Goal: Initialize repo, make commits, push to GitHub, create branch and pull request.

Steps:
1. mkdir devops-lab && cd devops-lab
2. git init
3. echo "# DevOps Lab" > README.md
4. git add README.md
5. git commit -m "initial commit"
6. Create GitHub repo → add remote → push:
   git remote add origin https://github.com/<user>/devops-lab.git
   git branch -M main
   git push -u origin main
7. Feature branch & PR:
   git checkout -b feature-1
   echo "change" >> README.md
   git add README.md
   git commit -m "feature-1 update"
   git push -u origin feature-1
   Create Pull Request in GitHub UI and merge.
2. Docker Containers and Images
Goal: Build a Docker image for a Python script and run it.

hello.py:
print('Hello from Docker!')

Dockerfile:
FROM python:3.11-slim
WORKDIR /app
COPY hello.py .
CMD ["python", "hello.py"]

Commands:
docker build -t hello-python:1.0 .
docker run --rm hello-python:1.0
3. Flask App Containerization and Docker Hub Deployment
Goal: Containerize a Flask app and push to Docker Hub.

app.py:
from flask import Flask
app = Flask(__name__)
@app.route('/')
def home(): return '<h1>Flask App</h1>'
if __name__=='__main__': app.run(host='0.0.0.0', port=5000)

Dockerfile:
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install flask
EXPOSE 5000
CMD ['python','app.py']

Commands:
docker build -t <dockerhub-user>/flask-lab:1.0 .
docker run -p 5000:5000 <dockerhub-user>/flask-lab:1.0
docker login
docker push <dockerhub-user>/flask-lab:1.0
4. CI Pipeline Lab (Python → GitHub Actions → Docker Hub)
Goal: Use GitHub Actions to build/push image.

.github/workflows/docker-publish.yml:
name: CI
on: push:
  branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USER }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - uses: docker/build-push-action@v4
      with:
        push: true
        tags: ${{ secrets.DOCKERHUB_USER }}/flask-lab:latest
5. Jenkins Deployment Using Docker
Goal: Jenkins pipeline builds & pushes Docker image.

Jenkinsfile:
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'docker build -t $DOCKERHUB_USER/flask-lab:jenkins .' } }
    stage('Push') { steps { withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) { sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'; sh 'docker push $DOCKERHUB_USER/flask-lab:jenkins' } } }
  }
}
6. Manage Kubernetes Resources Using CLI
Goal: Use kubectl to manage deployments.

Commands:
minikube start
kubectl apply -f k8s-deploy.yaml
kubectl get pods
kubectl get svc
kubectl logs <pod>
kubectl delete -f k8s-deploy.yaml

Sample deployment (k8s-deploy.yaml):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - name: flask
        image: <dockerhub-user>/flask-lab:1.0
        ports:
        - containerPort: 5000
7. Kubernetes Deployment and Service for Python App
Goal: Deploy image & expose via service.

Service YAML (k8s-svc.yaml):
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask
  ports:
  - port: 80
    targetPort: 5000
  type: NodePort

Commands:
kubectl apply -f k8s-deploy.yaml
kubectl apply -f k8s-svc.yaml
kubectl get svc
minikube service flask-service
