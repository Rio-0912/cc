# cc

## Exp 1 
1. Pull an Nginx image and run a container on port 8080.
   ```bash
   docker pull nginx
   docker run -d -p 8080:80 --name mynginx nginx
   http://localhost:8080 open broswer
   ```
3. Create a Dockerfile- Apache Server host the sample website with expose the
port,” then build and run it.
```bash
mkdir apache-site
cd apache-site

```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Apache Lab</title>
</head>
<body>
    <h1>Hello from Apache Docker Container</h1>
</body>
</html>
```

docker file 
```yaml
FROM httpd:latest

COPY index.html /usr/local/apache2/htdocs/

EXPOSE 80

docker build -t myapache .
docker run -d -p 8090:80 --name apachecontainer myapache
```

5. Show all running containers and remove a stopped container.
6. Use a bind mount to share a host folder with a container and verify file visibility.
`docker run -it --name bindtest -v C:\docker-share:/data ubuntu bash`


# Exp 2
Sure. Here’s your clean **MD file content for Exp 2 to Exp 10**. No fluff, just commands and what you need to survive your lab viva without crying later.

---

# cc

## Exp 2 – Docker Compose

### 1. Flask + Redis using docker-compose

```yaml
version: "3"

services:
  web:
    image: python:3.10
    working_dir: /app
    volumes:
      - .:/app
    command: bash -c "pip install flask redis && python app.py"
    ports:
      - "5000:5000"
    depends_on:
      - redis

  redis:
    image: redis:latest
```

### Flask app (app.py)

```python
from flask import Flask
import redis

app = Flask(__name__)
r = redis.Redis(host='redis', port=6379)

@app.route('/')
def home():
    r.incr('hits')
    return f"Hits: {r.get('hits').decode()}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### 2. Run stack + logs

```bash
docker compose up -d
docker compose logs -f
```

### 3. Scale Flask service

```bash
docker compose up -d --scale web=2
```

### 4. Stop & remove stack

```bash
docker compose down
```

---

## Exp 3 – Ansible

### 1. Inventory file

```ini
[local]
localhost ansible_connection=local
```

### 2. Playbook (install nginx)

```yaml
- name: Install Nginx
  hosts: local
  become: yes

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
```

### 3. Run playbook

```bash
ansible-playbook -i inventory playbook.yml
```

### 4. Ad-hoc disk usage

```bash
ansible local -i inventory -m command -a "df -h"
```

---

## Exp 4 – Kubernetes (Minikube)

### 1. Start cluster

```bash
minikube start
minikube status
```

### 2. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f deploy.yml
```

### 3. Expose NodePort

```bash
kubectl expose deployment nginx-deploy --type=NodePort --port=80
```

### 4. Scale

```bash
kubectl scale deployment nginx-deploy --replicas=3
kubectl get pods
```

---

## Exp 5 – AWS EC2 & Security Groups

```bash
aws ec2 run-instances \
--image-id ami-xxxxx \
--count 1 \
--instance-type t2.micro \
--key-name mykey \
--security-group-ids sg-xxxx
```

Allow SSH:

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxx \
--protocol tcp --port 22 --cidr 0.0.0.0/0
```

Install web server:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
```

Allow HTTP:

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxx \
--protocol tcp --port 80 --cidr 0.0.0.0/0
```

---

## Exp 6 – AWS S3

```bash
aws s3 mb s3://my-bucket-123
aws s3 cp file.txt s3://my-bucket-123/
aws s3api put-bucket-versioning \
--bucket my-bucket-123 \
--versioning-configuration Status=Enabled
```

Make public:

```bash
aws s3api put-object-acl \
--bucket my-bucket-123 \
--key file.txt \
--acl public-read
```

---

## Exp 7 – AWS VPC

Create VPC + subnets:

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

Public subnet:

```bash
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.1.0/24
```

Private subnet:

```bash
aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.2.0/24
```

Allow frontend → backend (DB port):

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-backend \
--protocol tcp --port 3306 --source-group sg-frontend
```

---

## Exp 8 – OpenStack Instance

```bash
openstack image list
openstack flavor list
```

Create instance:

```bash
openstack server create \
--image ubuntu \
--flavor m1.small \
--key-name mykey \
--security-group default \
my-instance
```

Floating IP:

```bash
openstack floating ip create public
openstack server add floating ip my-instance <ip>
```

---

## Exp 9 – Docker Registry

Build image:

```bash
docker build -t myapp .
```

Tag:

```bash
docker tag myapp username/myapp:latest
```

Push:

```bash
docker push username/myapp:latest
```

Pull:

```bash
docker pull username/myapp:latest
```

Scan:

```bash
docker scan username/myapp:latest
```

---

## Exp 10 – Kubernetes Advanced

### ConfigMap

```bash
kubectl create configmap app-config --from-literal=ENV=prod
```

### Secret

```bash
kubectl create secret generic mysecret \
--from-literal=password=12345
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Enable ingress:

```bash
minikube addons enable ingress
kubectl apply -f ingress.yml
```

### HPA

```bash
kubectl autoscale deployment nginx-deploy --cpu-percent=50 --min=1 --max=5
```

---

