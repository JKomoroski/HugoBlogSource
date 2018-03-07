---
title: "Let's Talk Kubernetes Ingress"
date: 2018-03-05T20:10:31-07:00
tags: [kubernetes, homelab, config]
draft: false 
---

If you don't know what [Kubernetes](https://kubernetes.io/) is, I highly suggest to look into it. It's a container orchestration system that runs anywhere. In short, it takes the simple but powerful concept of a containerized application (popularized by [docker](https://www.docker.com/what-docker)), and ramps it up to web scale levels. It allows you to deploy and manage hundreds or thousands of contaners without the pain of managing the VM and OS layers assocated with a normal
deployment. But the purpose of this post is not to preach container orchestration. My goal here is to get more tactical about the topic and talk about a specific component of Kubernetes called an Ingress Controller.

When I first started tinkering with my cluster, the most difficult piece of understanding what was going on was networking. Getting traffic into my cluser was not obvious from the start. There are some simple things you can set up like NodePort services, or if you're on a cloud provider, you may have access to LoadBalancer services. Since [my lab](/2018/02/20/my-lab/) is in my basement, I started with simple NodePort services. 

These worked, especially given the learning curve needed for them. They gave me an easy win where I could get started without having to learn everything. Below is an example Nginx Replication Controller, accessable via a node port service:

```
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  # Number of pods to run
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      # What container(s) will run in each pod
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    name: nginxservice
  name: nginxservice
spec:
  ports:
      # The port that this service should serve on within the cluster
    - port: 80
      # The destination port that traffic will go to at the pod 
      targetPort: 80
      # The port that the service will be accessed from outside the cluster at the node's ip
      nodePort: 31000
  # Label keys and values that must match in order to receive traffic for this service.
  selector:
    app: nginx
  type: NodePort
```

This sort of thing works, but if you want to route traffic directly to the cluster, you'd need to route traffic to a non-standard port. Also, what if you want to host multiple sites from your cluster, it would require you to stand up a reverse proxy to address. This is where ingress controllers enter. If you want to route based on path, or a host header ingress rules and a controller can make this very easy.

There are several applications that can act as a kubernetes ingress controller, Nginx, HAProxy, Rancher, Traefik and many more can all serve this purpose. In my lab, I'm runing [Traefik](https://traefik.io/) as my ingress controller. I like that it's a very small containter with all the bells and whistles. To deploy it to my lab, I just needed to apply the following file to my cluster:

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: traefik-daemonset
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: https
          containerPort: 443
          hostPort: 443
        - name: admin
          containerPort: 8080
        securityContext:
          privileged: true
        args:
        - --api
        - --kubernetes
```

This sets up the role, role binding, account and the daemonset for the application. With that, you would be ready to start setting up ingress rules, and routes as you see fit. In my cluster, I want to be able to see the Traefik web ui. So Lets set up an ingress and service to the Traefik application itself. 

```
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.mydomain.tld
    http:
      paths:
      - backend:
          serviceName:  traefik-web-ui
          servicePort: 80
```

This defines a service for the traefik UI (serice port 80, container port 8080). It then defines an ingress rule that the controller will use to define an inbound route. In this case it's a reverse proxy rule, that takes the inbound traffic with a host header "traefik.mydomain.tld" and sends it to the traefik-web-ui on port 80. If you have a dns record (or hostfile entry) that points your browser to the cluster host IP, you'll get the following:

<img src="/img/traefik_ingress_ui.png" alt="Traefik_UI" style="width: 750px;"/>

And there you have it. A quick intro to Kubernetes Ingress!
