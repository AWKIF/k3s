# k3s
lightweight install of k8s + istio with haproxy on docker-compose

## Prerequisites

You need docker & docker-compose  

Get kubectl  
```curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/linux/amd64/kubectl```  
```chmod +x ./kubectl```

## Install

Checkout this repo, then  

```
➜  k3s git:(master) ✗ docker-compose up -d   
Creating network "k3s_default" with the default driver
Creating volume "k3s_k3s-server" with default driver
Creating k3s_server_1 ... done
Creating k3s_node_1   ... done
Creating k3s_lb_1     ... done
```

Export the newly created kubeconfig.yaml file in the current directory  
```export KUBECONFIG=~/k3s/kubeconfig.yaml```

Wait 30s for cluster to start  
```
➜  k3s git:(master) ✗ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
45ba8ac31f2a   Ready     worker    4m        v1.14.3-k3s.1
```



then install istio  (it can take a while to pull istio docker images)

```➜  k3s git:(master) ✗ kubectl apply -f istio-k3sb.yaml```

It's done, you should have istio and kubernetes up and running:  
```
➜  k3s git:(master) ✗ kubectl get po --all-namespaces        
NAMESPACE            NAME                                      READY     STATUS      RESTARTS   AGE
istio-system         istio-citadel-58fc8bfb55-9zx45            1/1       Running     0          2m
istio-system         istio-cleanup-secrets-1.1.8-vt975         0/1       Completed   0          2m
istio-system         istio-galley-75bcd59768-vljbk             1/1       Running     0          2m
istio-system         istio-ingressgateway-d6c9b9fc-ws5zl       1/1       Running     0          2m
istio-system         istio-pilot-6d69f9c474-9zj5x              2/2       Running     0          2m
istio-system         istio-policy-6465bfd765-7nd7l             2/2       Running     1          2m
istio-system         istio-security-post-install-1.1.8-4grc4   1/1       Running     1          2m
istio-system         istio-telemetry-dbc4d76ff-cd672           2/2       Running     1          2m
istio-system         prometheus-d8d46c5b5-2v57c                1/1       Running     0          2m
kube-system          coredns-695688789-jvwjn                   1/1       Running     0          5m
local-path-storage   local-path-provisioner-69fc9568b9-xwfqc   1/1       Running     0          5m
```

## Deploy & test helm

Download helm:  
```curl -L https://git.io/get_helm.sh | bash``` should work or:  https://helm.sh/docs/using_helm/#installing-the-helm-client  

Then run:  
```helm init --service-account tiller```  

Install your app then add the domain to your ```/etc/hosts```, ie:    
```127.0.0.1	localhost mytestk3s.deez.re```  

You should now be able to check your app in your browser, ie:  
http://mytestk3s.deez.re/

## Troubleshooting

If you see istio pods in status ```ContainerCreating``` it usually means docker images have not been totally pulled yet.  
You can describe the ```istio-security-post-install-1.1.8-XXX``` pod (replace with your pod name) to check if it's the case:  
```Normal  Pulling    2m40s  kubelet, d1c46b516291  Pulling image "docker.io/istio/kubectl:1.1.8"```

