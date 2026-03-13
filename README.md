# using-ssl-certification-
Get EC2 public IP:

curl ifconfig.me

Example:

3.110.45.20-- this is your ec2 ip 

Generate certificate:

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout tls.key \
-out tls.crt \
-subj "/CN=3.110.45.20"--- replace this with your ip
6️⃣ Create TLS Secret   


kubectl create secret tls demo-tls \
--cert=tls.crt \
--key=tls.key

Verify:

kubectl get secrets
7️⃣ Create Ingress with TLS
ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  tls:
  - secretName: demo-tls

  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-service
            port:
              number: 80

Apply:

kubectl apply -f ingress.yaml
8️⃣ Get Minikube Node IP

Run:

minikube ip

Example:

192.168.49.2

But since you're using EC2 public IP, traffic will flow:

Internet → EC2 Public IP → Minikube → Ingress → Pod
9️⃣ Open EC2 Security Group

Allow ports:

80
443
🔟 Access Application

Open browser:

https://EC2_PUBLIC_IP

Example:

https://3.110.45.20

You will see Nginx demo page via HTTPS.

Browser will show warning because certificate is self-signed.

1️⃣1️⃣ Test Using Curl
curl -k https://EC2_PUBLIC_IP

Example output:

Welcome to nginx!
