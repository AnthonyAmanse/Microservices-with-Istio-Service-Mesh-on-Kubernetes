# Istio BookInfo

[Istio](http://istio.io) is an open platform that provides a uniform way to connect, manage, and secure microservices. Istio supports managing traffic flows between microservices, enforcing access policies, and aggregating telemetry data, all without requiring changes to the code of your microservice. Istio provides an easy way to create this service mesh by deploying a [control plane](https://istio.io/docs/concepts/what-is-istio/overview.html#architecture) and injecting sidecars, an extended version of the  [Envoy](https://lyft.github.io/envoy/) proxy, in the same Pod as your microservice.

The [BookInfo](https://istio.io/docs/samples/bookinfo.html) is a simple application that is composed of four microservices. The application is written in different languages for each of its microservices namely Python, Java, Ruby, and Node.js.

# Prerequisite

# Deploy to Bluemix

# 1. Installing Istio in your Cluster
## 1.1 Download the Istio source
  1. Download the latest Istio release for your OS: [Istio releases](https://github.com/istio/istio/releases)  
  2. Extract and go to the root directory.
  3. Copy the `istioctl` bin to your local bin  
  ```bash
  $ cp bin/istioctl /usr/local/bin
  ## example for macOS
  ```

## 1.2 Grant Permissions  
  1. Run the following command to check if your cluster has RBAC  
  ```bash
  $ kubectl api-versions | grep rbac
  ```  
  2. Grant permissions based on the version of your RBAC  
    * If you have an **alpha** version, run:

      ```bash
      $ kubectl apply -f install/kubernetes/istio-rbac-alpha.yaml
      ```

    * If you have a **beta** version, run:

      ```bash
      $ kubectl apply -f install/kubernetes/istio-rbac-beta.yaml
      ```

    * If **your cluster has no RBAC** enabled, proceed to installing the **Control Plane**.

## 1.3 Install the [Istio Control Plane](https://istio.io/docs/concepts/what-is-istio/overview.html#architecture) in your cluster  
```bash
kubectl apply -f install/kubernetes/istio.yaml
```
* _(Optional) For more options/addons such as installing Istio with [Auth feature](https://istio.io/docs/concepts/network-and-auth/auth.html) and [collecting telemetry data](https://istio.io/docs/tasks/metrics-logs.html), go [ here](https://istio.io/docs/tasks/installing-istio.html#prerequisites)._

# 2. Inject Istio Envoys on BookInfo Application
Envoys are deployed as sidecars on each microservice. Injecting Envoy into your microservice means that the Envoy sidecar would manage the ingoing and outgoing calls for the service. To inject an Envoy sidecar to an existing microservice configuration, do:
```bash
$ kubectl apply -f <(istioctl kube-inject -f samples/apps/bookinfo/bookinfo.yaml)
```
# 3. Access your Application
```bash
echo $(kubectl get po -l istio=ingress -o jsonpath={.items[0].status.hostIP}):$(kubectl get svc istio-ingress -o jsonpath={.spec.ports[0].nodePort})
184.xxx.yyy.zzz:30XYZ
```
`http://184.xxx.yyy.zzz:30XYZ/productpage`
# 4. Modify Service Routes
* Set Default Routes to `reviews-v1` for all microservices  
This would set all incoming routes on the services (indicated in the line `destination: <service>`) to the deployment with a tag `version: v1`. To set the default routes, run:
```bash
$ istioctl create -f samples/apps/bookinfo/route-rule-all-v1.yaml
```
* Set Route to `reviews-v2` of **reviews microservice** for a specific user  
This would set the route for the user `jason` to see the `version: v2` of the reviews microservice. Run:
```bash
$ istioctl create -f samples/apps/bookinfo/route-rule-reviews-test-v2.yaml
```
* Route 50% of traffic on **reviews microservice** to `reviews-v1` and 50% to `reviews-v3`.  
This is indicated by the `weight: 50` in the yaml file.
```bash
$ istioctl replace -f samples/apps/bookinfo/route-rule-reviews-50-v3.yaml
```
* Route 100% of the traffic to the `version: v3` of the **reviews microservicese**  
This would set every incoming traffic to the version v3 of the reviews microservice. Run:
```bash
$ istioctl replace -f samples/apps/bookinfo/route-rule-reviews-v3.yaml
```
