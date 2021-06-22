# k3s-servicelb-demo
How to quickly take advantage of the klipper-lb deployed out of the box with k3s

## Assumptions

You have successfully installed k3s and did not remove the service load balancer. For example:

`curl -sfL https://get.k3s.io | sh -`


## Contents

* `deployment.yaml`

    Contains the k8s spec for an nginx deployment

* `ing.yaml`

    Contains the k8s spec for exposing the service defined in svclb.yaml to the outside world

* `rancher-demo.yaml`

    Contains an alternate workload to nginx

* `service.yaml`

    Contains the k8s spec to expose the nginx deployment on a NodePort

* `svclb.yaml`

    Contains the k8s spec for a `LoadBalancer` type Service

## Steps

### Deploy the workload with a load balancer

1. `cd k8s-configs`
1. `kubectl apply -f deployment.yaml`
1. `kubectl apply -f svclb.yaml`
1. `kubectl apply -f ing.yaml`
1. Visit the ip address of your cluster to see the workload


### Deploy a NodePort service

1. If you already did this for the lb, you don't need to do it again

    `kubectl apply -f deployment.yaml`

1. kubectl apply -f service.yaml

### Check outcomes

* Since we deployed into the default namespace you can check with

    `kubectl get all`

* The output will look similar to this

    ```
    NAME                          READY   STATUS    RESTARTS   AGE
    pod/nginx-6c6c9b96c-jmpbb     1/1     Running   0          3m46s
    pod/nginx-6c6c9b96c-v4z9h     1/1     Running   0          3m46s
    pod/svclb-nginx-svclb-pqmqw   1/1     Running   0          113s

    NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
    service/kubernetes    ClusterIP      10.43.0.1       <none>          443/TCP          90d
    service/nginx-svclb   LoadBalancer   10.43.112.233   192.168.8.231   8080:30324/TCP   113s
    service/hello-world   NodePort       10.43.31.126    <none>          80:30001/TCP     12s

    NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/svclb-nginx-svclb   1         1         1       1            1           <none>          113s

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx   2/2     2            2           3m46s

    NAME                              DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-6c6c9b96c   2         2         2       3m46s
    ```

## Try these things

* Scale up the deployment

    `kubectl scale --replicas=5 deployment/nginx`

* Change the path in the ingress from `/` to `/nginx`

    * Now you can visit the ip of your cluster with a path to the specific app

* Deploy the `rancher-demo.yaml` for a visual look into load balancing

    * Create a service load balancer and ingress separately for this workload
    * Use a different path than you used for nginx
    * *NOTE* the port exposed on this workload is `8080` 
        * Be sure to update the targetPort in your service load balancer. 

