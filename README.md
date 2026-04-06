kubectl get all -n webapp
NAME                          READY   STATUS    RESTARTS   AGE
pod/webapp-69fc54f7fd-854sb   1/1     Running   0          22m
pod/webapp-69fc54f7fd-xsmdt   1/1     Running   0          22m

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/webapp-service   ClusterIP   10.103.200.141   <none>        80/TCP    22m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   2/2     2            2           22m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-69fc54f7fd   2         2         2       22m
ghada@ghada:~/k8s-app-deployment$
