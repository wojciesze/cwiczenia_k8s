1. Utwórz plik: `40-change-index.sh` o następującej zawartości:
```shell
#!/bin/sh

sed -i 's/index\.html/index.png/' /etc/nginx/conf.d/default.conf

```
2. Utwórz config mape:
```shell
kubectl create cm nginx-addon --from-file 40-change-index.sh
```
4. Do swojego namespace wrzuć następujący deployment, podmiejając `[trener] Moj pierwszy QR kod` na coś innego. Zapisz do dowolnego pliku i potem: `kubectl apply ...`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: qr-app
  name: qr-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: qr-app
  template:
    metadata:
      labels:
        app: qr-app
    spec:
      imagePullSecrets:
      - name: docker-conf
      volumes:
      - name: empty-vol
        emptyDir: {}
      - name: nginx-addon-vol
        configMap:
          name: nginx-addon
          defaultMode: 0755
      initContainers:
      - image: python:3
        name: qr-gen
        command:
        - sh
        - -c
        - |
          pip install pillow qrcode
          qr "[trener] Moj pierwszy QR kod" > /qr/index.png
        volumeMounts:
        - name: empty-vol
          mountPath: /qr
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: empty-vol
          mountPath: /usr/share/nginx/html
        - name: nginx-addon-vol
          mountPath: /docker-entrypoint.d/40-change-index.sh
          subPath: 40-change-index.sh
```
5. Upewnij się, że deployment działa
6. Utwórz serwis do swojego deploymentu
```shell
kubectl expose deployment qr-app --port=80 --target-port=80
```
7. A teraz ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    haproxy.org/ssl-redirect: "true"
    ingress.class: haproxy
  name: qr-app
spec:
  rules:
  - host: <username>.at.exitcode.xyz
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: qr-app
            port:
              number: 80
```
Zapisz do pliku: `ingress.yaml`, zmień <username> na swojego użytkownika i wrzuć na kubernetesa.

8. Upewnij sie, że obiekt typu `ingress` jest dostępny:
```shell
kubectl get ingress qr-app
```
9. W swojej przeglądarce otówrz URL: `http://<username>.at.exitcode.xyz`

