## 1️⃣ Install NGINX Ingress Controller on Minikube

Minikube makes this easy:

```bash
# Enable the ingress addon
minikube addons enable ingress
```

Verify:

```bash
kubectl get pods -n kube-system | grep ingress
```

You should see something like `nginx-ingress-controller` running. ✅

---

## 2️⃣ Update your local hosts file

We will use a friendly hostname: `node-react.local`

Add this line to your `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
<minikube_ip> node-react.local
```

Get Minikube IP:

```bash
minikube ip
```

Example:

```
192.168.49.2 node-react.local
```

---

## 3️⃣ Create Ingress Manifest: `ingress-dev.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dev-ingress
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: node-react.local
      http:
        paths:
          # Frontend route
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
          # Backend API route
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 4000
          - path: /api/
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 4000
```

✅ Features:

* All traffic to `/` → **frontend-service**
* All traffic to `/api` or `/api/*` → **backend-service**
* Friendly hostname `node-react.local`
* Uses **NGINX ingress controller**

---

## 4️⃣ Apply the ingress

```bash
kubectl apply -f ingress-dev.yaml
```

Check:

```bash
kubectl get ingress -n dev
```

You should see:

```
NAME          CLASS    HOSTS                 ADDRESS         PORTS   AGE
dev-ingress   nginx    node-react.local     192.168.49.2   80      1m
```

---

## 5️⃣ Verify routing

* Open browser: [http://node-react.local](http://node-react.local) → **frontend** should load
* Frontend calls `/api/...` → requests go to **backend**
* Backend still talks internally to **Postgres** via ClusterIP service

---
