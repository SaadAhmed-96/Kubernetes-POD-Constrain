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
                
The below yaml snippet of the webserver deployment has podAntiAffinity and podAffinity configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label app=store. This will also ensure that each web-server replica does not co-locate on a single node.


        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: web-server
        spec:
          selector:
            matchLabels:
              app: web-store
          replicas: 3
          template:
            metadata:
              labels:
                app: web-store
            spec:
              affinity:
                podAntiAffinity:
                  requiredDuringSchedulingIgnoredDuringExecution:
                  - labelSelector:
                      matchExpressions:
                      - key: app
                        operator: In
                        values:
                        - web-store
                    topologyKey: "kubernetes.io/hostname"
                podAffinity:
                  requiredDuringSchedulingIgnoredDuringExecution:
                  - labelSelector:
                      matchExpressions:
                      - key: app
                        operator: In
                        values:
                        - store
                    topologyKey: "kubernetes.io/hostname"
              containers:
              - name: web-app
                image: nginx:1.16-alpine
                
                
The above example uses PodAntiAffinity rule with topologyKey: "kubernetes.io/hostname" to deploy the redis cluster so that no two instances are located on the same host.
