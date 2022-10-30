# Service mesh and Istio
This repository will contain istio demos, examples and more.

## Pre-requisites.
- Kubernetes/Openshift cluster or local cluster/container cluster with [minikube](https://minikube.sigs.k8s.io/docs/start/) or [kind](https://kind.sigs.k8s.io/)
- Istio command line tool - to download and install kindly follow [that link](https://istio.io/latest/docs/setup/getting-started/#download)

## Procedures
1. Connect to Cluster(Or create cluster using kind or minikube), for example, if you're using  minikube, then to create/start the cluster:
```shell
minikube start
```

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

11. Print the endpoint of the service with the gateway address and port, and open it in a web browser to see that the web page shown correctly, or run the second command to do it automatically: 
```shell
echo "http://$GATEWAY_URL/productpage"
xdg-open http://$GATEWAY_URL/productpage
```

## Visualization Of Mesh - Kiali Dashboard 

In order to visualize your service mesh using Kiali dashboard, Istio integrates with several different telemetry apps, such as Prometheus(Time Series Metrics Aggregator) , Grafana(Metrics Visualization) And jaeger(Traces and spans Server):

Installs Kiali with all Telemtry add-ons:
 
1. Download manifests of kiali and addons from Istio repository 
```shell
git clone --sparse --filter=blob:none --depth=1 --branch=master https://github.com/istio/istio.git
cd istio
git sparse-checkout add samples/addons
cd ..
```
2. Install kiali and add-ons:
```shell
 kubectl apply -f istio/samples/addons
```

3. Wait for kiali Deployment to complete its rollout successfully
```shell
kubectl rollout status deployment/kiali -n istio-system
Waiting for deployment "kiali" rollout to finish: 0 of 1 updated replicas are available...
deployment "kiali" successfully rolled out
```

4. On a new terminal window, open kiali dashboard:
```shell
istioctl dashboard kiali
```
5. Simulate incoming traffic to your sample mesh
```shell
for i in {1..500}; do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done
```

6. If not opened automatically, open kiali dashboard site manually 
```shell
xdg-open http://localhost:20001/
```

7. At kiali site, set above namespace to `default`, and go to graph on the left panel, you should see your mesh visualization, if not shown, repeat step 5 again.





