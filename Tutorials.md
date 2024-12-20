### How to Display the End User’s IP Address Within Pods (Inside the Cluster)
To achieve this, you need to enable Proxy Protocols.

Edit the Ingress service by adding the following annotation and patch:
```
kubectl annotate svc ingress-nginx-controller -n ingress-nginx  service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol="true"
kubectl patch svc ingress-nginx-controller -n ingress-nginx --patch  '{"spec": {"externalTrafficPolicy": "Local"}}'
```

Update the Ingress ConfigMap by adding the following to the data section:
```
data:
  use-proxy-protocol: "true"
  use-forward-headers: "true"
  compute-full-forward-for: "true"
```

Restart the Ingress deployment:
```
kubectl rollout restart deployment ingress-nginx-controller -n ingress-nginx
```

If you’re installing Ingress from scratch using Helm, use the following command:
```
helm install nginx-ingress stable/nginx-ingress \
 --set controller.publishService.enabled=true \
 --set controller.config.use-forward-headers=true \
 --set controller.config.compute-full-forward-for=true \
 --set controller.config.use-proxy-protocol=true  \
 --set controller.service.annotations."service\.beta\.kubernetes\.io/do-loadbalancer-enable-proxy-protocol=true"
```
This setup ensures that the end user’s IP address is visible within pods inside the cluster, for example, in the Nginx logs.
