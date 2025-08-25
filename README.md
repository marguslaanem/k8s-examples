# Things to consider when deploying applications

## ArgoCD

Argo applications must use the designated argocd namespace: team-name-apps

## Namespaces  

- All namespaces created must start with a prefix 'app-'
- Strong suggestion is to keep namespace creation separate from the applications to prevent accidental deletion of all the objects in a namespace.

## Certificates  

In order to get automatic certificates from vault to your ingress you need to add the following annotations to the ingress object:

```
  cert-manager.io/cluster-issuer: vault-issuer
  cert-manager.io/private-key-algorithm: "ECDSA"
  cert-manager.io/private-key-size: "384"
```

Also the tls section of ingress must include the fqdn of the ingress and a secret name to assign to the certificate.

## Deployments  

All deployments and statefulsets must include a nodeselector. Currently 'role: autoscaling' and 'role: apps' are usable.

## Secrets

TODO

## Networking

By default all created namespaces get a default deny ingress/egress rule. This is done using Cilium Network policies
In order to access your app or access other apps. Users must set their own rules to allow connections from namespace to namespace.
Example rule to allow access from ingress controller:

```
---
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "allow-from-ingress"
  namespace: app-xxx
spec:
  endpointSelector:
    matchLabels: {}
  ingress:
    - fromEndpoints:
        - matchLabels:
            "k8s:io.kubernetes.pod.namespace": haproxy
```
