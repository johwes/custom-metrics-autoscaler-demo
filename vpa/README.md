## 1. Deploy demo application

```
$ oc create deployment simple-webpage --image=quay.io/jwesterl/simple-webpage --port=8080
$ oc expose deployment/simple-webpage
$ oc expose service/simple-webpage
```

## 2. Install operator
In the OpenShift Container Platform web console, click Operators → OperatorHub.

Choose VerticalPodAutoscaler from the list of available Operators, and click Install.

On the Install Operator page, ensure that the Operator recommended namespace option is selected. This installs the Operator in the mandatory openshift-vertical-pod-autoscaler namespace, which is automatically created if it does not exist.

Click Install.
