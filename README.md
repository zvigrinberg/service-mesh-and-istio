# Service mesh and Istio
This repository will contain istio demos, examples and more.

## Pre-requisites.
- Kubernetes/Openshift cluster or local cluster/container cluster with [minikube](https://minikube.sigs.k8s.io/docs/start/) or [kind](https://kind.sigs.k8s.io/)
- Istio command line tool - to download and install kindly follow [that link](https://istio.io/latest/docs/setup/getting-started/#download)

## Procedures
1. Connect to Cluster.
2. Install istio control planne on cluster
```shell
istioctl install --set profile=demo -y
```
3. Enable istio automatic sidecar injection to all pods in $NAMESPACE
```shell
export $NAMESPACE=default
kubectl label $NAMESPACE istio-injection=enabled
```
4. Install a sample/demo deployment
```shell
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.15/samples/bookinfo/platform/kube/bookinfo.yaml
```

5. Wait until all pods are up and running(both application container and sidecar container for each), whenever condition met, press CTRL+C
```shell
kubectl get pods -w
```

6. Deploy an Istio Gateway and an Istio VirtualService in order to enable ingress traffic into the k8s services of the demo application:
   
```shell
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

7. Make sure that everything is fine
```shell
istioctl analyze
✔ No validation issues found when analyzing namespace: default.
```

8. Create in a new seperated terminal, a local minikube tunnel which all data and traffic will pass through it to the istio Ingress Gateway:
```shell
minikube tunnel
```

9. Extract the ingress gateway host and port and display all extracted values:
```shell
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
env | grep -E 'INGRESS_|SECURE_IN'

```

10. Compose the Ingress Gateway address + port into env variable:
```shell
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

11. Print the endpoint of the service with the gateway address and port, and open it in a web browser to see the web page shown correctly, or run the second command to do it automatically: 
```shell
echo "http://$GATEWAY_URL/productpage"
xdg-open http://$GATEWAY_URL/productpage
```

