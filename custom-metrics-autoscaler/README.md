# Demo of Red Hat Custom Metrics Autoscaler Operator

*Disclaimer: This repository is maintained by the author and is in no way official supported instructions from Red Hat.*

Red Hat Custom Metrics Autoscaler is based on the upstream open source [Kubernetes Event-driven Autoscaling] project.  This demo steps the user through installing and configuring Custom Metrics Autoscaler operator and provides an example application to show scaling in action.

## Prerequisites

- An OpenShift 4 cluster updated to a current release which provides the Custom Metrics Autoscaler operator available in OperatorHub
- Logged in as a user account with `cluster-admin` role

## 1. Deploy demo application

```
$ oc new-project my-project
$ oc create deployment simple-webpage --image=quay.io/jwesterl/simple-webpage --port=8080
$ oc expose deployment/simple-webpage
$ oc expose service/simple-webpage
```

## 2. Install operator

From the Administrator perspective -> OperatorHub, install the Custom Metrics Autoscaler operator in the recommended `openshift-keda` namespace

Create a `KedaController` instance.  Default configuration values are fine.

## 3. Configure application for scaling

Create a serviceaccount named thanos, add the `cluster-monitoring-view` clusterrole with project scope, and determine its token secret:
```
$ oc project my-project
$ oc create sa thanos
$ oc adm policy add-cluster-role-to-user cluster-monitoring-view -z thanos
$ oc create secret generic thanos-bt-secret --from-literal=bearerToken=$(oc create token thanos --duration=8760h)
```

Create a project scoped [TriggerAuthentication] using the definition provided and replacing the token secret with your own.  You can alternately use a cluster scoped ClusterTriggerAuthentication (not covered in this demo).
```
$ oc apply -f resources/triggerauthentication.yaml
```

Create a [ScaledObject] using the definition provided.  Note that the trigger in this definition is configured with the [Prometheus scaler].  See the upstream Keda docs for a [complete list of scalers].  Also, the Prometheus trigger uses the bearer token for authentication and references the TriggerAuthentication we just created.
```
$ oc apply -f resources/scaledobject.yaml
```

## 4. Scaling our application

Now let's generate some application load.  Open another terminal and run the following, substituting the route name with your own:

```
URL=$(oc get route simple-webpage -o jsonpath='{.spec.host}')
while true; do curl -I http://$URL; done
```

The Prometheus query that was used is `sum(rate(haproxy_backend_connections_total{route="httpd-example"}[1m]))`, 

Let's observe the application scaling real time.  From the Administrator perspective -> Observe -> Metrics, use `sum(rate(haproxy_backend_connections_total{route="simple-webpage"}[1m]))` as the expression and press Run queries.  This was the query provided in the ScaledObject and represents the number of concurrent haproxy backend connections for the `simple-webpage` route, averaged over the past 2 minutes.  For those stepping through this demo and using [OpenShift's built-in monitoring stack], this is a perfect place to try various Prometheus queries and determine relevant metrics for your own application.

  The threshold of this query is set to '1' in the demo.  In other words, based on the number of concurrent haproxy backend connections over the past 2 minutes, it will spin up another pod replica for every count of 1.  From the top right, change the refresh to every 15 seconds and observe the value over time.

From your terminal, watch the application pod replicas:
```
$ watch oc get pods -n my-project
```

Note that underneath the Custom Metrics Autoscaler is a `HorizontalPodAutoscaler` definition:
```
$ oc get hpa -n my-project
```

Finally, you can also view the events to determine the resources in this demo were set up properly and scaling is actively taking place.
```
$ oc get events -n my-project
```

## License

GPLv3

## (Original) Author

Kevin Chung

[Kubernetes Event-driven Autoscaling]: https://keda.sh/
[TriggerAuthentication]: resources/triggerauthentication.yaml
[ScaledObject]: resources/scaledobject.yaml
[Prometheus scaler]: https://keda.sh/docs/2.9/scalers/prometheus/
[complete list of scalers]: https://keda.sh/docs/2.9/scalers/
[OpenShift's built-in monitoring stack]: https://docs.openshift.com/container-platform/4.12/monitoring/monitoring-overview.html
