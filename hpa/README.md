## Intro
This short README shows how to deploy a simple app and use the HorizontalPodAutoscaler (HPA) to automatically scale the deployment up or down depending on utilization.
The official documentation can be found here, [Openshift Horizontal Pod Autoscaler Documentation]
## 1. Deploy demo application

```
oc new-project my-project
oc create deployment simple-webpage --image=quay.io/jwesterl/simple-webpage --port=8080 --replicas=2
oc expose deployment/simple-webpage
oc expose service/simple-webpage
oc set resources deployment simple-webpage --limits=cpu=500m,memory=512Mi --requests=cpu=50m,memory=256Mi
```

## 2. Create a "HorizontalPodAutoscaler"
Create the HPA object 
(There is also an example vpa configurationf for a DeploymentConfig)
```
oc apply -f hpa.yaml
```

## 3. Check status of HPA and POD's
You can check on the POD's and the HPA with the following commands

```
oc describe <POD>
oc describe hpa hpa-cpu-simple-webapp
```

## 4. Simulate some load to trigger automatic scaling
Open a shell inside the pod (instead of "oc rsh" you can also open the Openshift UI and go to the terminal of the pod)
Then simulate some load

```
oc rsh <POD>
dd if=/dev/urandom | b2sum - > /dev/null
```

leave the terminal / shell running the dd command and open another terminal to inspect the HPA object and scaling
```
watch "oc get pods"
oc describe hpa hpa-cpu-simple-webapp
```

As a last step CTRL+c the dd command in the pod and you should see a scale down after a while.

[Openshift Horizontal Pod Autoscaler Documentation]: https://docs.openshift.com/container-platform/latest/nodes/pods/nodes-pods-autoscaling.html

