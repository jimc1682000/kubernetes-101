取得官方的nginx
```
docker pull nginx:1.17.10-alpine
```

啟動 Container
```
docker run -d -p 8000:80 --name nginx_test nginx:1.17.10-alpine
```

確認頁面正常
```
curl http://127.0.0.1:8000
```

建立nginx-deployment.yaml, deployment will create a pod
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.10-alpine
        ports:
        - containerPort: 80

```
Note: imagePullPolicy: Never means it will use local image, and will not pull images from docker.io.

建立django-servce.yaml, service will expose port to external network, which we can setup at nodePort
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30670
```

佈署deployment and service
```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml

kubectl get pods
kubectl get deployment
kubectl get service
``

確認服務正常
```
curl http://192.168.1.113:30670
```

清除服務
```
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-deployment.yaml
```