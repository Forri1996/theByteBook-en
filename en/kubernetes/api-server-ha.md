# API Server High Avaliable Load Balance

In a Kubernetes cluster, the apiserver serves as the entry point for the entire cluster. Any user or program intending to perform operations such as creating, deleting, modifying, or querying cluster resources needs to go through kube-apiserver. Consequently, its high availability determines the overall high availability capability of the entire cluster.


The kube-apiserver is essentially a stateless server. To achieve high availability, multiple instances of kube-apiserver are typically deployed, along with the introduction of an external load balancer (referred to as LB below) for traffic proxying. Subsequent users (such as kubectl, dashboard, and other clients) and components within the cluster will access the apiserver through the LB.