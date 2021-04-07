先參考DJANGO-README.md建立DJANGO環境

建立nginx folder
```
mkdir nginx
cd nginx
```

建立nginx.conf
```
# the upstream component nginx needs to connect to
upstream django {
    # server unix:///opt/uwsgi/fms-uwsgi.sock; # for a file socket
    server localhost:7001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen 80;

    # serve domain name
    server_name 192.168.1.113;  # substitute your machine's IP address or FQDN
    charset utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /usr/share/nginx/html/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /usr/share/nginx/html/static; # your Django project's static files - amend as required
    }

    # send all non-media requests to the Django server.
    location / {
        uwsgi_pass django;
        include /etc/nginx/uwsgi_param; # the uwsgi_params file you installed
    }
}
```

從下列網站中複製uwsgi_param
https://github.com/nginx/nginx/blob/master/conf/uwsgi_params

修改settings.py
```
import os

STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

執行指令產生static files
```
python manage.py collectstatic
```

建立Dockerfile
```
FROM nginx:1.17.10-alpine

# nginx.conf
ENV WORKER_PROCESS_NUM=1
ENV SEND_FILE=off

COPY nginx/uwsgi_param /etc/nginx/
COPY nginx/mysite_nginx.conf /etc/nginx/conf.d/default.conf
COPY demo/static /usr/share/nginx/html/static
```

建立docker image
```
cd ..
docker build -t demo-nginx -f nginx/Dockerfile .
```

建立int-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: int-deployment
spec:
  selector:
    matchLabels:
      app: demo-nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: demo-nginx
    spec:
      containers:
      - name: nginx
        image: demo-nginx
        imagePullPolicy: Never
        ports:
        - containerPort: 80
      - name: django
        image: demo
        # command: ["uwsgi", "--socket", ":8000", "--plugin", "python", "--wsgi-file", "test.py"]
        imagePullPolicy: Never
        ports:
        - containerPort: 7000
          name: django-port
        - containerPort: 7001
          name: uwsgi-socket
```
Note: imagePullPolicy: Never means it will use local image, and will not pull images from docker.io.

建立int-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: int-service
spec:
  type: NodePort
  selector:
    app: demo-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30670
```

佈署deployment and service
```
kubectl apply -f int-deployment.yaml
kubectl apply -f int-service.yaml

kubectl get pods
kubectl get deployment
kubectl get service
``

確認服務正常
```
open http://192.168.1.113:30670/
open http://192.168.1.113:30670/static/admin/img/icon-no.svg
```

清除服務
```
kubectl delete -f int-service.yaml
kubectl delete -f int-deployment.yaml
```

REF:
https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html