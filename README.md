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

3 - deploy and expose the application as internal service. This will not make the application public yet.

```
kubectl apply -f deployment.yaml
```

5 - create a service for the pods created by the deployment during step 4.

```
kubectl apply -f service.yaml
```

6 - create a managed certificate resource specifying the domain the certificate will be created for.
NOTES:

- Only once certificate per domain can be created.
- Requires gke varsion 1.12.6 or higher

```
kubectl apply -f certificate.yaml
```

7 - define a static IP for the application. To use a static IP using Ingress the following conditions must be met:

- A Service with `type:NodePort`
- An Ingress configured with the service name and static IP annotation
- Global IP addresses only work with Ingress resource type

```
gcloud compute addresses create <static-ip-name> --global
```

8 - create ingress resource to generate a load balancer that will route traffic to the application. In the ingress manifest set

- the networking.gke.io/managed-certificates annotation to the name of your certificate.
- the kubernetes.io/ingress.global-static-ip-name annotation to the name of your reserved IP address.

See ingress.yaml > metadata.annotations.

```
kubectl apply -f ingress.yaml
```

9 - configure the domain DNS A Record to point to the global static IP address.
The load balancer IP address car be retrieved with

```
kubectl get ingress
```

10 - wait until the certificate is provisioned. The certificate status can be retrived with

```
kubectl describe managedcertificate
```

11 - [optional] clean up.

```
kubectl delete ingress <ingress-name>

gcloud compute addresses delete <static-ip-name> --global

gcloud container clusters delete <cluster-name>
```

## Useful links:

- https://cloud.google.com/kubernetes-engine/docs/tutorials/configuring-domain-name-static-ip
- https://cloud.google.com/load-balancing/docs/ssl-certificates
- https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl
- https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs
- https://github.com/GoogleCloudPlatform/gke-managed-certs
