# 12.Kubernetes.Data.Security

## Add index.html page as config map, which should display hostname of pod as first level header

### Config file - k8-3.yaml

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-map
data:
  test.html: |
    <HTML><HEAD><TITLE>A.Borisevich</TITLE></HEAD>
    <BODY><h1>ZDAROVO OTEC</h1></BODY></HTML>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: simple-web
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: "50%"
      maxUnavailable: "0%"
  selector:
    matchLabels:
      app: simple-web
  template:
    metadata:
      labels:
        app: simple-web
    spec:
      containers:
        - name: php-nginx
          image: ubuntu/nginx
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 50m
              memory: 50Mi
            limits:
              cpu: 100m
              memory: 100Mi
          volumeMounts:
            - name: tmp
              mountPath: /var/www/html/index.html
              subPath: index.html
            - name: ssh-keys
              readOnly: true
              mountPath: "/root/.ssh"
      initContainers:
        - name: init
          image: busybox
          command: [ "sh" ]
          args: [ "-c", "cd /var/www/html/; cat test.html > index.html;
          sed -i 's/ZDAROVO OTEC/Hello from $(MY_POD_NAME)/' index.html;
          cat index.html;
          cp /var/www/html/index.html /tmp/artifact/index.html" ]
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          volumeMounts:
            - name: config-map-mount
              mountPath: /var/www/html/test.html
              subPath: test.html
            - name: tmp
              mountPath: /tmp/artifact
      volumes:
        - name: tmp
          emptyDir: {}
        - name: config-map-mount
          configMap:
            name: config-map
        - name: ssh-keys
          secret:
            secretName: ssh-keys
            defaultMode: 0600
---
apiVersion: v1
kind: Service
metadata:
  name: simple-web-service
  labels:
    run: simple-web-service
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: simple-web

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-sa
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-alias: "nginx-test.k8s-3.sa"
spec:
  rules:
    - host: nginx-test.k8s-3.sa
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: simple-web-service
                port:
                  number: 80
```

### Validation of index.html

![Hello from first pod](kuber12.png)

![Hello from second pod](kuber12_1.png)

## Generate pair public and private keys (or use existing) and add them to the pod for user root as secrets

### sealed_secret.yaml

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: ssh-keys
  namespace: default
spec:
  encryptedData:
    id_rsa: AgDGtWWxQ+B8OYgNUhc2iuw7WyqMW2kinXlcYXks7jji/yJreO0pXl+S1dUgjy1OfBAjlm3/W0jNogjx6UCTAurPoYo+f1TAs8S50IMPWLPdVjs7jy/DHqmONGGAigNyVchhSTF6dfVZT5b6u6m7zrt1/TfuVgKJ34FAp6rSVMW1mai9JNagXRIZcgcDF5TSjTsDwuzC/s9U7gwJufkhp4gZ6UxkLCmMIOFZWrA6HNHt68qpmwpXc8m4I81f3kZMreUXOg7oKDa7in9ttP8JG+ECi5NEMjb6E1szZRiZhtu0M+18CQYN7w33g1Fq1X7hR6624P3rcK47RMJL9VpcX0mzia03OF+Aa2eUw9TG4reow8jBuWDfvGt4pf7hgRSOmZ0AiogXWw7Yp6HAz7R1iBjylpS15o9zk5mb+b1qqutKRsjVB8iUj/MLq6Ng6d9iYT+FkZWvEebjuwfuz0fruILmrSA+MxvQtV895S2T5N/76xO5lA6/oJzcLQ+iBn3joFDVE73WccslaflcgrGDWziQk7IlAbsGCI3iI9iKOCYOaY4rvsuPd7l34KhmCcEv9WsTvDAZgeKGvp/5GXX/0IKhhUdvlDyXjqKrYmwbiCVcPYIdDvRad7Ct8CGbXvZU/yqK+5hdm9DbkVDiQ8/CTytSpdwH7XVZ6/EkgYTV02lt8hNfkhoopbUm5B9JZfEUt/QB5NUyzCw5YwL9Fn42DhWNmKVJ9oyM+8S7uURI07xTb8n7f8sVulvvPb5H0FfhNnVo6ec8yWiz7WxefbUbqXrEZcvkg+X+r+1/ZBzHgoSsHOsekc95lRkm9KQ3n8kz6IzQGiQ/KR8ukxFKmDgBcrVLJFgMs98ezYOJBJ5f/TINteH8SdtQYl2ShhZO+OU3fbAm8F7O7y4Ytb8g1QC2mBB2o5CWo+hCTNvIwwN5d/xuQbTaRJPOjhKQoNtY7+YTIh65qHRBdpSHA6y3P2t4DMOV0MZi2K3kKbHRe6EYoN7Vou6tYUvP0PT0ZN+qUJKewae3avBkUd5SRKt98xSiroZn+JXwgj9LmafvKmGvC/jqaeKND/0ymBVWtXWTgdb9zReEDVJeY6Em/opENbdqZA+paWXgaIzlNQZB1GH7b9PuG8IpQNqtRtUR8y2LXOYMi+Jw/ejjinW8n0r9TrHuPlJWnGgafB623MTsnWVruPF6U9QhqHy7yJclRe+Qw03vPsb6/Vlu6U8ESURW2iQdJ/ZaW17ZekDpyHZ7FnNxhBP0ULojUI1s67P+dYHMfsCjpYxNXDc4XGdPurdY7HNl7LY8/6ghSoJDG+vZHEJlPkn4IvonmK3sOCsa/qEMIvOb8j6S1ulAkE9wtK7GoZzx7AM96/kTqlMI2/+UJFjYNV50BmJ39Vq8dXMxmY1FXBojzppZchPOE3U0J4Ss4/xQmKNpKpcOCCoqFfMdIz6w7fy9Ea1EBJDW1yOFJVBIO4uByKVyjg6MkiCZM8QQ6SVUXUwnMEmknM5EnL5oOPzlUuLtg04nBkHFxbEojPwYub6VuxhzIbsKYTTpVY/S9yQHwsOH7SLMECOIYcwCGUyVHsa+9hklBay+FEuGf+Ed0dX3jMpQxO9uCngI5GGC+oM74u14iT4RWUMJQfFrrQ+7dAkNsnX6IRbEnnQXggmhVGk8FQuUAc5Xavf625l8/nUfycy3F96d0aHtTcnSiyPt5YOmqELb3CV1SjtOS/HkknPqPvpS4SqQ5IA+G777DjxStexTtMG77op8CK9V/2yktARARncNS/HLnEqxkQZWc9U3L6sHf6txqUxhISniwXH97DKysbOVMDMnISD4JiSMuerkpvMJDa6+AwHv43i1LE57vxJFqD6g5KBhRdlIjrXPJfMbIPQFOtT6Fp+mP0RcwryafOdWLvtGAloCncdYmGdmxed1o3l6MB8lBZttDOgthRoF2bW9sWLJ0IKqEZ6trxA1IEaatkR4E0/SA/acanxukWX2ugzSudOXM0GmbE5+gWIj0ASj0ct+51uNHaB+pHX2wLSjKc9k5EWtnLkkoxTftSb5r74VYbt9dWh81A0zoZav4bfv2gQ9XU0v+TXQRCupM+rTLw4zWecVjX8lHWQB/hD8wg9OMDo+B5JMivLtMHxL5mh7vHxRIbWb4GRjXwyn52+NQEGrI01lgdyzu8tHriDv8+EBYIPTbhmjhc8jNj6QgfrNjdh0Zp3HrYFhio96EXvpvz0VYzRYE8+nH+oA1STSLNA839pVJtNIpYZwVQSCkcMEJ5bVTAJkhZ8iYOHqHDOrVdDZxKU2BpNKNvapNViDZrP/ZGUfvfKlF+sKkZAUDAAriggPXUPzBsZzPazFkLiyK2YHL7LEp4sG5hmbQ0uPDAK/G0qr4oTMcDeC75oIp2NNWGw5heA1vf5l1Bfs7k6CoqnCu61K8nITiMMBvQo5OijH5Aq2/r1bNWNmFTw0VyAki9YaCJ1zUnCYV5AsXAvd9jJRKceErWt4+K6fTpwMuoMnNjjkFewrYRCFpABTyewE+6PZ1NJWWifoa3kNleHAsT39OoQ+jr8iasla5clEVEgPlP0Xxt0KhM5+UHA7k55rX3WU/p5xOteq2PpNdotsh4ol8s4VWlvU1oeDr9xOpJpF7MeDjkKErydySGc/U2WmoqrZ2gwHfd/EVPoC6P1DUE1Fi8gNyIKxMdZ4iatlynzigTg4BI68lNZthfuT4ptpm/XrEEC6+pkk0ogiyYdAI4SkJ1giTrrUKjjWgdXL33/J5ISnI1uGOpMTk7m6lufYQHZeYvkc1nUeZ0mEVJAMwFHpsMGlbnr8qN7HA6UuYNvLXr4KwJU2KRuAbUzX4ku6o9LWfF+90PIWbEYgS5MT4XD6YCSVL7NPFZbReammoj8Zhdagw2s8thxKOLAaQvy/lvkLKO3qQNHilzCi7Xed7lf1Cg5I2MrV5QdIc2rJvMF4UP+u7x5TDq1fMCpQQC7/BPOEua7c5YO91K5dD0xoBNeqRGC6KOywTdB29O9VIpbO0LZiCr4dy+KCDEBByrMwsF0lUK+AAkj9jso0XMaYPsi50maQG2CBnbiSfKxdVHvnsUcMwrv7x2bW1Czn8oXbFp9/CeXQ/wtmge4HO+hMFuZQQTm0XgiySbN6rAqfMcWmmDIsG3x0nUwa9L+jsf9y5Sx9oggrAFQ4ObE/Qspc2a2D1uDiw2yyzlH+btz4gGcF9LEuvGHy8Ieuv32d3AwGwuTksxE20VNtoilldz1FZXzXB2DqcGCK86QUAAA9eMY2XnMMFN4PVUXFbE5gEBZhLSfVZlUVSdrOYt3+MwQhmWcll0m2XoJAGxDYdmch85syhD1324ASlWda9DCZmNl2MB6u09KLUGC5WTyP740FvBmzeprEgMwQP2NKcyPpNoz0ucfUbGrGhpGaIBZAKIv/VP8SqNOqQ/owNzCr+eTJ5CI61ok61wHqf645vZdoQGZsGfuMnSiW6NwwEpxznyhNhaD4woG8fY6bwP4qM1tmOc0MUo4i06BXzVxkqGn3+l1YbEp2Tx0h60E52Vr5jimX/ZZgqBxiImBb+SlnD/Ap77DTz2LvqsrABmJrl6ej0xsEfM/Ko75poJq9Fp1NrdO43OQf28gQMw4vQ6grYpnfo1lPh2zbO0dE/ZrrR5Y0GYQDyzibi2WpmJ/PIp2wklAKFrjmX945iPcIZDwQa5bh5xYaAVgrsSi6cfP8fy7j7xbA+ppzLmBjxcehF0l7T8QDsWsAcCmZW/k4i3SsCBt7wQNPLG/uwVuqRBjTC43d+R5DklmixT1INoqWV+rbN8rvVfK9TQ4q5X7qxDEWVusaM+H2oVTRsjK6eNiAZqymfkpXlVfQOhiNezimztgNQk8SpIsFjZaBxfFqpJVvJramuBlwMrKSTesdgyM2SnLSIC9hMDCNEqjWrUwP8FGZVhyGsWR/VVZH9rDSZjn2epGB0ktB6aF3CpL2kYaR+XKIj1zPOLW3Bm3SFx7EnMqOSHILY/b7DkrRiW6keqQlB19SerJbZIQCa206nJ0uq+Bgk7Sc9tiuRtmo2DJk4vR7BfIIX6GEl1rKawzCCWaVpi+RLvAiVOlvqmSFWDpg6U+nha8fBUAr98NuqEjM5TyOaObrmYAivBBcN+V4lHqprChtz0M/VjOmfEI8KiPvcMFKehFk+k49hp/F
    id_rsa.pub: AgBJJat1dsjeuMiOGcle0+qSntDPkTxjWFHLtGtlRt8UUzqSC3Ov9skDBi7iAwxiOqzqvrfLUs+73Eg7dlL9pw8pmi3q3qz4OC4MzcrV8sudJRXgpEp1KylrYbuSOq2Xu6UidwyUPCdoJ8Iq1kFs7Zz6MvR+2NsJX4HOQ6MmPUVh6nXPaHrQQS6c9nnkTcloq1lqeFxh5OnVdz0Yse7NEWKtvNo7608QhhqnzuGxb61LcCEQLzym+/rQXiSKjfinMRU7+/bm2tIGFYzl/UB4AYgZ41KF5pteGXlVA1VFCGc2mM2w81cHrcqxkut/CxqD5QNCCDkadNFEWV2YCd7fB+n2f/kKQYFzMutq6CO8rjbt+hYmhCS/Db8FPp/cDzrd1PknSazTUAlaj2NMxDF1j6gVj1WbDGvQYQm0MB6V//DwWgCyYRsP/nbUxv9eO1pOxXdT/88WhS16913lRCLGofsZ/dP4LnJWnGUjTuQe4aOEtmu1DYLsbppzwOmGkTL+eF2xdslx9OThZuvT9WK3BG4ldIIH4ctEAqJFV7WU6HfHS382jrxWK0tF4AEuo3Q3TqMYxDoTGH9TMRlM0oozEV0Ym+Olh8yCIlq3m+qdZwP4vDGXI0FICvBFLCSyaCpcsGvrUfvYW4QYHNPNpV7xbF6t1Z6PfOvKnVtzv9bObxt3sHnHYRdzt1QseWr95aOKdI7DPJOCwiRVCRhpa4H1WOqH9YzmcSvASfVC2jwIChRgOcazZlGSBibnW+IIfZjL1a6BMpvOf+8gyCAAE+Dut7uHKzmph4Wix1WB279Au49yx15gB2N3aLtR4nGikJT+tOnP8KiCVo5e2fXXy8dIFxA4whRuXioQPoa4ofVOlYfA/I/jKHQ/LxIldyJ71viyGPyLSGYtWOWgzYwRhA7Wy2iPM3f2/wkJ8gDpf8L2tDIGw0Lrxof61IG5Vcgc8QXQFa5H+23eXhbA2MKGqPqKkVCqY7K0wOJsjNXTogR8WJzqYuKTzg9B6DMBaF9pY7oFYhTpgndeoTG4evpwIjr7dAgwkZROs6YHXzVkzSJb3Whg0kTuHL5YfQJ/73ulnrqXstIonENz3p8I2b4TOfo3xSUVJmTEx0Sar+vMFB0BTT0nWWnFhduWn5sizlw6zXnxMm9Zz0lGbjxFsT5d2GUWoKjHcLkTkQ8wZDgmOMHd4OLochOzgP/NamemFOL/RXjpXvXah2y8L3QQAPDDmKa2PRC7adrwDcRNckfROSP30hTFz99TOuCR3huJrlosjdTZwxwdRlz6RNITr8h7gOOT4YTRpmznbPzKMAEon4S6xBn8+3SeN2B6KH6hxHuhCH+h3fqYvt3szXIZCdGNk+m4FxeqB13DnTCBnd2GVSrrcJAHbkx07tVNrH7/VgFaNBy+c4X3tMFi2dymR5hlVCVN6O6AeuZwlCiGWk+GIpC+pMfRD9KsQ74UK64bpQ==
  template:
    metadata:
      creationTimestamp: null
      name: ssh-keys
      namespace: default
    type: Opaque
```
### Validation of secret 

![Secret unsealed](secret_unsealed.png)

![Secret mounted](secrets_mounted.png)

![Public key](secret_public_key.png)


