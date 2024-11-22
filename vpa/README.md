## Intro
This short README shows how to deploy a simple app and use the VerticalPodAutoscaler (VPA) to either automatically apply resource requests (requests/limits) or have VPA only recommend requests/limits.

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

## 3. Create a "VerticalPodAutoscaler"
Create the VPA object using the recommender setting or Auto setting depending if you want the VPA controller to automatically change the resource requests of the pods or not.
```
oc apply -f vpa-recommender.yaml
OR
oc apply -f vpa-auto.yaml
```
