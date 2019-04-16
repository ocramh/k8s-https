# Kubernetes SSL Ingress and Service Deployment Setup on GCP

A simple k8s project that demonstrates how to set up a web service, route traffic to it via a load balancer and configure the load balancer to use SSL certificates.

1 - create container cluster. can also be done via gcp console.

```
gcloud container clusters create <cluster-name> --zone <zone-name> --machine-type g1-small --num-nodes 1
```

2 - retrieve cluster credentials if needed.

```
gcloud container clusters get-credentials <cluster-name>
```

3 - deploy the application.

```
kubectl run <deployment-name> --image=gcr.io/google-samples/hello-app:1.0 --port=8080
```

4 - expose deployment as internal service. This will not make the application public yet.

```
kubectl expose deployment <deployment-name> --target-port=8080 --type=NodePort

or

kubectl apply -f deployment.yaml
```

5 - create a service for the pods created by the deployment during step 4.

```
kubectl apply -f service.yaml
```

6 - create an SSL certificate and a set of keys.

```
// key
openssl genrsa -out test-ingress.key 2048

// signing request
openssl req -new -key test-ingress.key -out test-ingress.csr \
    -subj "/CN=k8s.shapes.ai"

// certificate
openssl x509 -req -days 365 -in test-ingress.csr -signkey test-ingress.key \
    -out test-ingress.crt
```

7a - Create a secret for holding certificate and key.

```
kubectl create secret tls my-secret \
  --cert test-ingress.crt --key test-ingress.key
```

7b - create ingress resource to generate a load balancer that will route traffic to the application and deploy it. In the ingress manifest specify to use the certificate generated in the previous step.

```
kubectl apply -f ingress.yaml
```

8a - [optional] define a static IP for the application. To use a static IP using Ingress the following conditions must be met:

- A Service with `type:NodePort`
- An Ingress configured with the service name and static IP annotation
- Global IP addresses only work with Ingress resource type

```
gcloud compute addresses create <static-ip-name> --global
```

8b - [optional] configure the existing Ingress resource to use the reserved IP address. See ingress.yaml > metadata.annotations.

8c - [optional] edit the domain dns A Record to point to the global static IP address.

NOTE: see https://cloud.google.com/load-balancing/docs/ssl-certificates#create-managed-ssl-cert-resource for instructions about how to set up a certificate signed by google.
To retrive the ingress proxy name run `gcloud compute target-https-proxies list`.

9 - [optional] clean up.

```
kubectl delete ingress <ingress-name>

gcloud compute addresses delete <static-ip-name> --global

gcloud container clusters delete <cluster-name>
```

## Useful links:

- https://cloud.google.com/kubernetes-engine/docs/tutorials/configuring-domain-name-static-ip
- https://cloud.google.com/load-balancing/docs/ssl-certificates
- https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl
