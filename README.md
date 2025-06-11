# ŚUU 2025 - Project: Istio-OTel
Team mebers: Anna Cichocka, Bartosz Rzepa, Piotr Sobczyński

## Introduction
Istio is a open source service mesh, that can be used for Kubernetes. Projects goal is to provide its case – study using OpenTelemetry framework. 

## Theoretical background/technology stack
### Istio
#### What is Istio
- open source service mesh
- transparent
- language-independent
- platform-independent (eg. Cloud, Kubernetes)
- used to organize microservices-based apps: secure, connect and monitor microservices
#### Istio’s selling points
- consistent service networking - manage services networking without developer overhead
- secure your services - service-to-service security including authentication, authorization and encryption
- improve performance - implement best practices (eg. canary rollouts) and get deep visibility into apps to identify where to focus your efforts to improve performance
#### Istio’s applications:	
- secure cloud-native apps - strong identity-based authentication, authorization and encryption
- manage traffic effectively - fine-grained control of traffic (rich routing rules, retries, failovers, and fault injection)
- monitor service mesh - allows deep understanding of how service performance impacts matters upstream (robust tracing, monitoring, logging)
- easily deploy with Kubernetes and virtual machines
- simplify load balancing - automated load balancing, advanced features like client-based routing and canary rollouts.
- enforce policies - pluggable policy layer and configuration API that supports access controls, rate limits, and quotas
#### Network traffic management in Istio:
- application level routing - traffic routing based on HTTP headers, URLs or services version (canary deployment i blue-green deployment)
- load balancing - traffic can be distributed across instances, many balancing strategies (round-robin, least connections, pasm)
- retries, timeouts
- circuit breaking - automatic cutting off unstable components and routing to alternative instances (to withstand services overload)
- testing tools:
- fault injection - failure simulation (delay, HTTP errors)
- traffic mirroring - mirroring existing requests for new services
#### Istio components:
- Istiod (Central Control Plane) - Istiod is responsible for service discovery, configuration management, and certificate distribution across the entire service mesh. It translates high-level routing rules into Envoy-specific configurations and dynamically distributes them to sidecar proxies at runtime. Istiod abstracts platform-specific service discovery mechanisms—such as those provided by Kubernetes—and converts them into a unified format that can be consumed by any sidecar proxy compatible with the Envoy API. On the security level, Istiod enables strong service-to-service and end-user authentication through built-in identity and credential management. It supports enforcing policies based on service identity rather than relying on less stable Layer 3 or Layer 4 network identifiers. Additionally, Istiod facilitates the transformation of unencrypted traffic within the mesh into secure, encrypted communication.
- Envoy proxy - mediates all inbound and outbound traffic for all services in the service mesh. Envoy proxies are the only Istio components that interact with data plane traffic. It implements the sidecar proxy model, which allows for adding Istio capabilities to an existing deployment without changing existing application code.  Envoy proxies enriches services with Envoy’s many built-in features, for example: Dynamic service discovery, Load balancing, TLS termination, HTTP/2 and gRPC proxies, Circuit breakers, Health checks, Staged rollouts with %-based traffic split, Fault injection, Rich metrics.
- Ingress and Egress Gateways - they are used to manage inbound and outbound traffic for your mesh, letting you specify which traffic you want to enter or leave the mesh. Gateway configurations are applied to standalone Envoy proxies that are running at the edge of the mesh, rather than sidecar Envoy proxies running alongside services.

### Open Telemetry
Open source observability framework. OTel is implemented in application itself, providing unified interface for observability. Supplies metrics, logs, traces for application. Data is platform – independent. Because of that many observability backends can be used without need to modify application code (eg. Prometheus). In order to display data from backend, observability frontend can be used (eg. Grafana)

### Kiali
Kiali is a management console for Istio service mesh with strong observability, configuration and validation capabilities. It provides a dashboard visualizing the mesh topology with rich information about inter-service dependencies and traffic health (latency, errors). Moreover, Kiali enables inspecting and producing service mesh configuration via a web interface and observing how the updated configuration affects performance of individual application services.

### Prometheus
Prometheus is an open-source monitoring and alerting tool designed for recording time-series metrics. It collects data by scraping HTTP endpoints from applications and stores it in its own time-series database. It’s widely used in cloud-native environments due to its reliability, scalability, and ease of integration. It uses a pull-based model, regularly scraping metrics endpoints exposed by instrumented applications. Prometheus has its own powerful query language called PromQL, allowing flexible data aggregation, filtering, and alerting. It comes with a built-in time-series database (TSDB) and an Alertmanager for sending notifications based on metric thresholds or conditions.  It integrates well with tools like Grafana for visualization.
Grafana
Grafana Open Source Software enables you to query, visualize, alert on, and explore your metrics, logs, and traces wherever they’re stored. Grafana data source plugins enable you to query data sources including time series databases like Prometheus and CloudWatch, logging tools like Loki and Elasticsearch, NoSQL/SQL databases like Postgres, CI/CD tooling like GitHub, and many more. Grafana provides you with tools to display that data on live dashboards with insightful graphs and visualizations. 


## Demo description
### Demo app:
Chosen demo comes from OpenTelemetry documentation and contains microservices written in different programming languages that communicate over gRPC and HTTP. It contains a load generator which fakes user traffic enabling the collection of metrics and traces by OpenTelemetry. The application will be improved by adding new versions of few services.


rescource: https://opentelemetry.io/docs/demo/architecture/

### Test case - canary deployment:
- create multiple versions for chosen microservices
- create DestinationRule and VirtualService policies for chosen microservices with canary deployment files
- observe changes in metrics, traces, logs
### Attributes across the system: [https://opentelemetry.io/docs/demo/telemetry-features/](https://opentelemetry.io/docs/demo/telemetry-features/)

## Solution architecture

## Environment configuration description

## Instalation method:
- install:
    - [istioctl](https://istio.io/latest/docs/setup/install/istioctl/)
    - [helm](https://helm.sh/docs/intro/quickstart/)
    - helpful for local deployment: [k9s](https://k9scli.io/)
```
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
```
```
minikube start
```
```
istioctl manifest apply --set profile=demo
```
```
kubectl label namespace default istio-injection=enabled
```
```
helm install my-otel-demo open-telemetry/opentelemetry-demo
```
```
kubectl --namespace default port-forward svc/frontend-proxy 8080:8080
```
### Making changes to configuration
- save configuration
```
helm show values open-telemetry/opentelemetry-demo > custom-values.yaml
```
- edit `cutom-values.yaml` file
- restart demo
```
helm uninstall my-otel-demo
```
```
helm install my-otel-demo open-telemetry/opentelemetry-demo -f custom-values.yaml
```
## How to reproduce

## Demo deployment steps
### Configuration setup
Whole setup configuration is held in `custom-values.yaml` file. Here's its overview:
- image: the official [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo) - microservice-based distributed Astronomy Shop,
- components: stripped-down to crucial ones, to allow for deployment on less-powerful machines (locally),
    - `opentelemetry-collector` - collects data across whole system, processes and exports it,
    - `jaeger` - allows for capturing traces,
    - `prometheus` - uses OpenTelemetry and Jaeger as data sources, provides querying capabilities, allows for data analysis
    - `grafana` - provides a platform for visualization and analysis data stored by Prometheus.
Canary deployment has been tested in a few load distribution setups.

### Data preparation

### Execution procedure

### Results presentation
Charts below show number of queries that take over 1s (green) and these that take less than 1s (orange)

#### Weights 0/100
![canary 0/100](/img/canary-0-100.png)
#### Weight 30/70
![canary 30/70](/img/canary-30-70.png)
#### Weight 50/50
![canary 50/50](/img/canary-50-50.png)
#### Weight 70/30
![canary 70/30](/img/canary-70-30.png)
#### Weight 100/0
![canary 100/0](/img/canary-100-0.png)


## Using AI in the project

## Summary - conclusion

## Bibliography:
https://cloud.google.com/learn/what-is-istio
https://boringowl.io/blog/istio-wprowadzenie-do-zarzadzania-uslugami-w-srodowisku-mikrouslug
https://www.youtube.com/watch?v=iEEIabOha8U

## TEMPORARY

<!-- Jeśli edycja zaszła w helm chart (custom_values.yaml) to należy stworzyć kubernetes manifesty na nowo -->
<!-- Po uruchomieniu tej komendy, trzeba ręcznie wyłączyć sidecar injection w Deployment jaeger (dodać spec:template:annotations: sidecar.istio.io/inject: "false") -->

make generate-kubernetes-manifests

<!-- komendy działają z tego folderu -->
cd opentelemetry-demo

<!-- Włączenie minikube -->
minikube start

<!-- Budowa cart service -->
eval $(minikube docker-env)
docker compose build cart

<!-- Instalacja istio w klastrze kubernetes -->
istioctl manifest apply --set profile=demo

<!-- kubectl label namespace default istio-injection=enabled - tego zdaje się nie trzeba bo jest w manifeście kubernetesowym na samym początku ale głowy za to nie dam -->

<!-- Deploy podstawowych mikroserwisów -->
kubectl create -f kubernetes/opentelemetry-demo.yaml

kubectl delete -f kubernetes/opentelemetry-demo.yaml

<!-- Deploy cart service -->
kubectl apply -f kubernetes/cart-deployment.yaml

kubectl delete -f kubernetes/cart-deployment.yaml

<!-- Destination rule i Virtual service dla cart dla canary deployment -->
<!-- Aby dostosować ruch pomiędzy wersjami wystarczy edytować weight w pliku opentelemetry-demo/kubernetes/cart-canary.yaml a następnie zastosować zmiany -->
kubectl apply -f kubernetes/cart-canary.yaml

kubectl delete -f kubernetes/cart-canary.yaml

<!-- Port froward aby mieć dostęp do narzędzi wewnątrz klastra -->
kubectl port-forward svc/frontend-proxy 8080:8080

<!-- Do benchmarku użyto komendy hey -->
go install github.com/rakyll/hey@latest

hey -n 1000000 -c 5 "http://localhost:8080/api/cart"


<!-- Metryki opentelemetry-collector -> prometeusz (query language) -> grafana
<!-- ilość zapytań powyżej 1s -->
sum(rate(traces_span_metrics_duration_milliseconds_bucket{le="+Inf",span_name="POST /oteldemo.CartService/GetCart"}[2m])) 
- 
sum(rate(traces_span_metrics_duration_milliseconds_bucket{le="1000",span_name="POST /oteldemo.CartService/GetCart"}[2m]))

<!-- Ilość zapytań poniżej 1s -->
sum(rate(traces_span_metrics_duration_milliseconds_bucket{le="1000",span_name="POST /oteldemo.CartService/GetCart"}[2m]))


<!-- Czyszczenie wszystkiego -->
kubectl delete -f kubernetes/cart-deployment.yaml
kubectl delete -f kubernetes/cart-canary.yaml
kubectl delete -f kubernetes/opentelemetry-demo.yaml
istioctl uninstall -y --purge
kubectl delete namespace istio-system
kubectl label namespace default istio-injection-

Zmiany zaszły w opentelemetry-demo/src/cart/src, dodano zmienną artificialDelayMs która jest zczytywana ze zmiennych środowiskowych, jest to sztuczne opóźnienie wszystkich metod CartService

<!-- Budowanie lokalnego serwisu cart -->
cd opentelemetry-demo
minikube start
eval $(minikube -p minikube docker-env)
docker compose build cart

