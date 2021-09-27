# Kubernetes-POD-Constrain
Understanding how we can constrain pod to a node using affinity/anti-affinity feature

## Use Case:
In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.

Here is the yaml snippet of a simple redis deployment with three replicas and selector label app=store . The deployment has PodAntiAffinity configured to ensure the scheduler does not colocate replicas on a single node.


        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: redis-cache
        spec:
          selector:
            matchLabels:
              app: store
          replicas: 3
          template:
            metadata:
              labels:
                app: store
            spec:
              affinity:
                podAntiAffinity:
                  requiredDuringSchedulingIgnoredDuringExecution:
                  - labelSelector:
                      matchExpressions:
                      - key: app
                        operator: In
                        values:
                        - store
                    topologyKey: "kubernetes.io/hostname"
              containers:
              - name: redis-server
                image: redis:3.2-alpine
                
