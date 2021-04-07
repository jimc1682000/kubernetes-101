建立 Django 專案
```
django-admin startproject demo
```

建立python requirement
```
cd demo
vim requirements.txt
    Django==3.1.2
```

python 虛擬環境
```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
deactivate
```

建立 uwsgi.ini
```
[uwsgi]

wsgi-file = ./demo/wsgi.py
; wsgi-file = ./test.py
socket = :7001
master = true
processes = 4
http-socket= :7000
restart= always
vacumum = true
virtualenv = ./venv
die-on-term = true
plugins = python3
```

建立 Dockerfile
```
FROM alpine:3.7
COPY . /demo
WORKDIR /demo
RUN apk add --no-cache \
        uwsgi-python3 \
        python3 && \
    pip3 install --no-cache-dir -r requirements.txt
EXPOSE 7000
CMD ["uwsgi", "-i", "uwsgi.ini"]
```

建立 Docker Image
```
docker build -t demo .
```

啟動 Container
```
docker run -d -p 7000:7000 --name demo_container demo
```

確認頁面正常
```
curl http://127.0.0.1:7000
```

建立django-deployment.yaml, deployment will create a pod
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-deployment
spec:
  selector:
    matchLabels:
      app: django
  replicas: 1
  template:
    metadata:
      labels:
        app: django
    spec:
      containers:
      - name: django
        image: demo
        imagePullPolicy: Never
        ports:
        - containerPort: 7000
```
Note: imagePullPolicy: Never means it will use local image, and will not pull images from docker.io.

建立django-servce.yaml, service will expose port to external network, which we can setup at nodePort
```
apiVersion: v1
kind: Service
metadata:
  name: django-service
spec:
  type: NodePort
  selector:
    app: django
  ports:
  - protocol: TCP
    port: 7000
    targetPort: 7000
    nodePort: 30770
```

佈署deployment and service
```
kubectl apply -f django-deployment.yaml
kubectl apply -f django-service.yaml

kubectl get pods
kubectl get deployment
kubectl get service
``

確認服務正常
```
curl http://192.168.1.113:7000
```

清除服務
```
kubectl delete -f django-service.yaml
kubectl delete -f django-deployment.yaml
```

REF:
https://medium.com/@bryant5968/%E9%80%8F%E9%81%8E-docker-%E5%BB%BA%E7%AB%8B-django-uwsgi-%E7%92%B0%E5%A2%83-78ae2edd9124
