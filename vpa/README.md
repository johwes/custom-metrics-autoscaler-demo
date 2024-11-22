## 1. Deploy demo application

```
$ oc new-project my-project
$ oc create deployment simple-webpage --image=quay.io/jwesterl/simple-webpage --port=8080
$ oc expose deployment/simple-webpage
$ oc expose service/simple-webpage
$ oc set resources deployment simple-webpage --limits=cpu=2,memory=512Mi --requests=cpu=1,memory=256Mi
```

## 2. Install operator
In the OpenShift Container Platform web console, click Operators → OperatorHub.

Choose VerticalPodAutoscaler from the list of available Operators, and click Install.

On the Install Operator page, ensure that the Operator recommended namespace option is selected. This installs the Operator in the mandatory openshift-vertical-pod-autoscaler namespace, which is automatically created if it does not exist.

Click Install.

## 3. Create a "VerticalPodAutoscaler" object using the recommender setting
```
oc apply -f vpa.yaml
```
