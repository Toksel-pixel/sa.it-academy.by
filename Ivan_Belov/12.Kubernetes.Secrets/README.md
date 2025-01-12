#### 12.Kubernetes. Secrets
#
#
#

#### Report

```bash
---
apiVersion: v1
kind: Namespace
metadata:
  name: homework-12

---
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: ssh-keys
  namespace: homework-12
spec:
  encryptedData:
    id_private: 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
    id_pub: AgC+cNKLZC2z86jZeoGJUB1ujI3PGnXegf7vL6v4uYtYNsnYsyEM+egz0lCkBazUpbEbmXfH9epEkHk26nnnyCRkUAoTU/8RoBaTRFbxsfqwfgBHSFNX1fJDXIV6cSuOpy4ps6CnxkRwWBieOYGqZneO/qETTQyG8H9pOd9Upx1pP6WtExaC4jkAIQi+W/9/QeTM7ywVhD+gP5NyUMolu4OTvKTIUgumWcwH6N31GMVYO81NwNkb9hUGjOT2w34t8Cj0lNa1XYUZsTBfTtpkS/cITo1Tfw9V1MnLC2b8PvsgP0qN1S2pxeOrCmp/biGP6pwEO9wfyX5+1DbQGy6mU1cC6YFsyei003B5DffR0pl/ewERdX6z0HAMMXvdW1EhYtohN2Y2kab6ArDfk6mh4qV3fJswQ5ELvUbyNPDKyao4P5hPFPmw59s1h7LRQcCrt9zM4zZmAC56a9PtYQJ1esSGk9tYkup0OrnYUrBe9tVv4X6L+jfMoV9kCktaRG0aVulKdgYFdrdlyUA3WU1WArpVJ6LOPgZ/5Jlim3LdoCxPk9/1X7yXUroK6wy67V9QZWJUBchKLfWjjXGxLwxQYRDRSTHBDIxi0c66fv6xCWjJKbqFaeINvDGTFEQCe3f7ZRHvVYboaiUTOQSgUr2gO2Fxmu2Gh6qxyEOB37qXk8HhL+cbVTyov/AfoIzAtssxrZ0z8Vb6ByxNNb/mHqTVV4he/kvAtV84vGFwkCotv9moT2nfB/N6kFhlvPRfenWymFBEWQosyqtPm5baBCj30ey9OcLUTUXb9tzXGNV/1g/gz44eEQFLfO/MzwX722q1AwLpYzC4mogndyPf3Uu4NIoH+2oUOhSmrJQalTC2Zf4SW7b5sfBj2xYEdCDm1NQxa/v8ztWw+px4nzjopzMFQaO3xSbTKLAxzRxtpFzZIKUgKLGD7YgZKcKgNMFQX6WMdwm8j5y3ZM9qdsMb2JH29DkWBCPbDyHGibWItlpvJIFcwU+c3urenNLLEdPbshxTyR1SaOc+JSTrChMoafQ0GAbiIK9HlrdIg5qEzeVXHzWq7QEpDXPuVn/46KMyBzId2ULbznEOsm+pCu8nbSH6b2d8Kg5hn3BKT27v+iO21DdUm4WN1bpfcrTlLqDARDuoCGsXJM+NoRerPZUhvEkAYWfToKxvpU4vlWZzC044gr4QCxWx2WTxSoNwuYErYCqBEG9QCSWhv7FvCTwZ0ILJLqWpf7C42hWgmkonVk4LC86s7vhEVEB3lN3hnr/91gRVDhhcR8DBA21b2KsDpYF/lvQsxfxQyNl4e5ck338YxutwytyuHpjGuH5qe6x60YvWe24QZVXPgqX6Oq1PJ0tMwIfMju0ZzBGfh2/lruhpaatDK6kQvcJasH/ZMmnQbXtumuTq9LAsOc0A3taJ2ELreBqLoESUD8k6zsKUIh4mR3xofVsGbqiyw1RE
  template:
    metadata:
      creationTimestamp: null
      name: ssh-keys
      namespace: homework-12

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
  namespace: homework-12
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3 # tells deployment to run 2 pods matching the template
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 33%
      maxUnavailable: 0%
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.22.1
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html
        - name: ssh-keys
          mountPath: /root/.ssh/
          

      initContainers:
      - name: init
        image: busybox:1.28
        command: ['sh', '-c']
        securityContext:
          readOnlyRootFilesystem: false
        args:
          - echo "<html><head>HOSTNAME INFO</head><body><h1>$HOSTNAME</h1></body></html>">/usr/share/nginx/html/index.html
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html
          readOnly: false
      volumes:
      - name: config
        emptyDir: {}
      - name: ssh-keys
        secret:
          secretName: ssh-keys
          defaultMode: 0600


---
apiVersion: v1
kind: Service
metadata:
  namespace: homework-12
  labels:
    app: nginx-service
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: homework-12
  name: ingress-sa
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-alias: "web.k8s-2.sa"
spec:
  rules:
    - host: app.k8s-2.sa
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
![CURL](HW12KEYS.png)
![CURL](12HW.png)
