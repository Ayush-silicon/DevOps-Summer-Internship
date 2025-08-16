npm create vite@latest to create the application

actions -> setup your own workflow

```yaml

name: Test and Build

on:
  push:
    branches:
    - main
    paths:
    - '**/*'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    #Setting up environment
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Docker Setup
      uses: docker/setup-buildx-action@v2

    - name: Docker Credentials
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Docker tag
      id: version
      run: |
        VERSION=v$(date +"%Y%m%d%H%M%S")
        echo "VERSION=$VERSION" >> $GITHUB_ENV

    # Build the Docker Image
    - name: Build Docker Image
      run: |
        docker build -t sibasish934/argocd-demo-frontend:${{ env.VERSION }} .

    # Push the Docker Image
    - name: Push Docker Image
      run: |
        docker push sibasish934/argocd-demo-frontend:${{ env.VERSION }}

    # UPdate the K8s Manifest Files
    - name: Update K8s Manifests
      run: |
        cat deploy/deployment.yaml
        sed -i "s|image: sibasish934/argocd-demo-frontend:.*|image: sibasish934/argocd-demo-frontend:${{ env.VERSION }}|g" deploy/deployment.yaml
        cat deploy/deployment.yaml

    # Update Github
    - name: Commit the changes
      run: |
        git config --global user.email "<>"
        git config --global user.name "GitHub Actions Bot"
        git checkout main
        git add deploy/deployment.yaml
        git commit -m "Update deploy.yaml with new image version - ${{ env.VERSION }}"
        git push origin main

```

settings -> secrets and variables -> create repo secrets

actions -> general -> workflow permission -> read and write permission. 


Commands to install and login to argocd 

```shell

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=testing-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --version 1.13.0
