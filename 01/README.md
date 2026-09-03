# Solution 01

## Task 01

1. Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: task01
  labels:
    app: task01
spec:
  replicas: 1
  selector:
    matchLabels:
      app: task01
  template:
    metadata:
      labels:
        app: task01
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - name: http
              protocol: TCP
              containerPort: 80
            - name: https
              protocol: TCP
              containerPort: 443
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
        - name: network-multitool
          image: wbitt/network-multitool
          env:
            - name: HTTP_PORT
              value: "8080"
            - name: HTTPS_PORT
              value: "8443"
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

2. Command `kubectl scale --replicas=2 deployment/task01`
3. Log:

    ```sh
    azabelin@ubuntu01 01 % kubectl scale --replicas=2 deployment/task01
    deployment.apps/task01 scaled
    azabelin@ubuntu01 01 % kubectl get pod
    NAME                      READY   STATUS    RESTARTS   AGE
    task01-57564bf9c6-dc45t   2/2     Running   0          5m39s
    task01-57564bf9c6-pv87h   2/2     Running   0          7s
    azabelin@ubuntu01 01 % kubectl get deployment task01
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    task01   2/2     2            2           7m15s
    azabelin@ubuntu01 01 % kubectl get deployment task01 -o wide
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                IMAGES                          SELECTOR
    task01   2/2     2            2           7m23s   nginx,network-multitool   nginx,wbitt/network-multitool   app=task01
    azabelin@ubuntu01 01 % kubectl scale --replicas=3 deployment/task01
    deployment.apps/task01 scaled
    azabelin@ubuntu01 01 % kubectl get deployment task01 -o wide
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                IMAGES                          SELECTOR
    task01   3/3     3            3           7m47s   nginx,network-multitool   nginx,wbitt/network-multitool   app=task01
    azabelin@ubuntu01 01 % kubectl get pod
    NAME                      READY   STATUS    RESTARTS   AGE
    task01-57564bf9c6-dc45t   2/2     Running   0          7m55s
    task01-57564bf9c6-f4phf   2/2     Running   0          22s
    task01-57564bf9c6-pv87h   2/2     Running   0          2m23s
    azabelin@ubuntu01 01 %
    ```

4. Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: task01
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
  selector:
    app: task01
  type: ClusterIP
```

5. Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: network-check
spec:
    restartPolicy: Never
    containers:
        - name: multitool
          image: wbitt/network-multitool
          command: ["/bin/sh", "-c"]
          args:
            - echo "=== Check connection to Nginx ===" && curl -iv --connect-timeout 5 http://task01:80
```

## Task 02

1. Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: task02
  labels:
    app: task02
spec:
  replicas: 1
  selector:
    matchLabels:
      app: task02
  template:
    metadata:
      labels:
        app: task02
    spec:
      initContainers:
        - name: busybox
          image: busybox
          command: ["/bin/sh", "-c"]
          args:
            - |
              until nslookup task02.default.svc.cluster.local; do echo "waiting for nginx-service..."; sleep 2; done
      containers:
        - name: nginx
          image: nginx
          ports:
            - name: http
              protocol: TCP
              containerPort: 80
            - name: https
              protocol: TCP
              containerPort: 443
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

2. Log:

```sh
azabelin@ubuntu01 ~ % kubectl get pod     
NAME                     READY   STATUS     RESTARTS   AGE
task02-75d7d5bcc-8l6d9   0/1     Init:0/1   0          17s
azabelin@ubuntu01 ~ % kubectl logs task02-75d7d5bcc-8l6d9 -c busybox
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find task02.default.svc.cluster.local: NXDOMAIN

** server can't find task02.default.svc.cluster.local: NXDOMAIN

waiting for nginx-service...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find task02.default.svc.cluster.local: NXDOMAIN

** server can't find task02.default.svc.cluster.local: NXDOMAIN

waiting for nginx-service...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find task02.default.svc.cluster.local: NXDOMAIN

** server can't find task02.default.svc.cluster.local: NXDOMAIN

waiting for nginx-service...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find task02.default.svc.cluster.local: NXDOMAIN

** server can't find task02.default.svc.cluster.local: NXDOMAIN

waiting for nginx-service...
```

3. Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: task02
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
  selector:
    app: task02
  type: ClusterIP
```

4. Log:

```sh
azabelin@ubuntu01 ~ % kubectl get pod                               
NAME                     READY   STATUS    RESTARTS   AGE
task02-75d7d5bcc-8l6d9   1/1     Running   0          2m22s
azabelin@ubuntu01 ~ % 
azabelin@ubuntu01 ~ % kubectl describe pod task02-75d7d5bcc-8l6d9
Name:             task02-75d7d5bcc-8l6d9
Namespace:        default
Priority:         0
Service Account:  default
Node:             desktop-control-plane/172.18.0.2
Start Time:       Fri, 04 Sep 2026 06:37:31 +0700
Labels:           app=task02
                  pod-template-hash=75d7d5bcc
Annotations:      <none>
Status:           Running
IP:               10.244.0.23
IPs:
  IP:           10.244.0.23
Controlled By:  ReplicaSet/task02-75d7d5bcc
Init Containers:
  busybox:
    Container ID:  containerd://0c4f1292365a5e72eb30cde94a4cd677393c67138255e57ed6b2cc85a52e5a37
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:dc2d74b28e4cf8984fa52af1f39bc7c3d9c73760b41a74d629f5d11b1ab28616
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
    Args:
      until nslookup task02.default.svc.cluster.local; do echo "waiting for nginx-service..."; sleep 2; done
      
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 04 Sep 2026 06:37:33 +0700
      Finished:     Fri, 04 Sep 2026 06:39:38 +0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cj7m8 (ro)
Containers:
  nginx:
    Container ID:   containerd://55c135cb84a6aac11f1da831eac12310c733f49f51c428e9c0222d847cfa0374
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:05b8cb60c354a44ab824ea6e7dc69b46d50762cdbe728a347a5b656e6fb3d7c4
    Ports:          80/TCP (http), 443/TCP (https)
    Host Ports:     0/TCP (http), 0/TCP (https)
    State:          Running
      Started:      Fri, 04 Sep 2026 06:39:41 +0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cj7m8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-cj7m8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  6m48s  default-scheduler  Successfully assigned default/task02-75d7d5bcc-8l6d9 to desktop-control-plane
  Normal  Pulling    6m48s  kubelet            spec.initContainers{busybox}: Pulling image "busybox"
  Normal  Pulled     6m46s  kubelet            spec.initContainers{busybox}: Successfully pulled image "busybox" in 2.199s (2.199s including waiting). Image size: 1925780 bytes.
  Normal  Created    6m46s  kubelet            spec.initContainers{busybox}: Container created
  Normal  Started    6m46s  kubelet            spec.initContainers{busybox}: Container started
  Normal  Pulling    4m41s  kubelet            spec.containers{nginx}: Pulling image "nginx"
  Normal  Pulled     4m38s  kubelet            spec.containers{nginx}: Successfully pulled image "nginx" in 2.899s (2.899s including waiting). Image size: 65050320 bytes.
  Normal  Created    4m38s  kubelet            spec.containers{nginx}: Container created
  Normal  Started    4m38s  kubelet            spec.containers{nginx}: Container started
azabelin@ubuntu01 ~ % 
```