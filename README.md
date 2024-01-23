# Create-Cert-manager-ClustterIssuer-with-Cloudflare
![image](https://github.com/Hamzachelligue/Create-Cert-manager-ClustterIssuer-with-Cloudflare/assets/6747298/0b016583-a581-43a2-b8ed-e48cdbb719e4)

In this project, I will explain how to connect your Cloudflare account to Cert-Manager in order to generate and manage free TLS certificates for your website from a recognized authority.

Before starting you should have :
- A running kubernetes cluster V 1.25 or higher
- A CloudFlare account

## 1 - Installing and Configuring Cert-Manager.

```console
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml
```
## 2 - Create a Cloudflare token with specific authorization. 

 My Profile (Right top corner) > API Tokens. Click Create Token button and Create custom token button. And then, fill Permission section form below.

![image](https://github.com/Hamzachelligue/Create-Cert-manager-ClustterIssuer-with-Cloudflare/assets/6747298/9f682927-257d-4d68-b264-a273ccfddde9)

## 3 - Create your ClustterIssuer with Cloudflare DNS proof

```
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token
  namespace: cert-manager
type: Opaque
stringData:
  api-token: <Cloudflare API token>
----
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: <Issuer Name>
spec:
  acme:
    email: <Your Email>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: cluster-issuer-account-key
    solvers:
    - dns01:
        cloudflare:
          email: <Your Email>
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
```
Finally, you can create Ingress with Auto Create Let’s Encrypt certificate using the example yaml below. In this example, I use nginx-ingress.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-cloudflare
  annotations:
    cert-manager.io/cluster-issuer: "cloudflare"
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - <your_domain.com> (should be added on cloudflare)
    secretName: your-domain-tls
  rules:
  - host: <your_domain.com>
    http:
        paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: <YOUR_SERVICE_NAME>
              port:
                number: 80
```
