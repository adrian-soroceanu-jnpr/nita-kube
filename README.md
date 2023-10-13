Porting of Juniper NITA project to Kubernetes

The setup of the infrastructure pods is straigth forward, at the end of the setup we should have 4 running pods that are:
```
kubectl get pods
NAME                      READY   STATUS    RESTARTS       AGE
db-6878b6446-gk6rs        1/1     Running   5 (3d1h ago)   52d
jenkins-577757858-jqvg5   1/1     Running   0              4h56m
proxy-6d75b768bc-kpdpr    1/1     Running   7 (3d1h ago)   52d
webapp-67d64dbb99-pfrb9   1/1     Running   0              25h
```
The pv.yaml and pv2.yaml are creating persistent volumes for Jenkins pods and MariaDB, this persisten volumes are claimed by jenkins-home-persistenvolumeclaime.yaml and mariadb-persistentvolumeclaim.yaml
```
jcluser@ubuntu:~$ kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pv-volume        2Gi        RWO            Retain           Bound    default/mariadb        manual                  52d
task-pv-volume   20Gi       RWO            Retain           Bound    default/jenkins-home   manual                  52d
```
```
jcluser@ubuntu:~$ kubectl get pvc
NAME           STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
jenkins-home   Bound    task-pv-volume   20Gi       RWO            manual         52d
mariadb        Bound    pv-volume        2Gi        RWO            manual         52d
```
The jenkins-deployment.yaml, db-deployment.yaml, proxy-deployment.yaml, webapp-deployment.yaml are used to spin the actual deployments of the containers
```
jcluser@ubuntu:~$ kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
db        1/1     1            1           52d
jenkins   1/1     1            1           5h21m
proxy     1/1     1            1           52d
webapp    1/1     1            1           25h
```
The db-service.yaml, jenkins-service.yaml, proxy-service.yaml, webapp-service are used to expose ports between the applications and also to the outside world.
```
jcluser@ubuntu:~$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
db           ClusterIP   10.109.37.245   <none>        3306/TCP            52d
jenkins      ClusterIP   10.104.250.46   <none>        8443/TCP,8080/TCP   52d
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP             52d
proxy        ClusterIP   10.98.2.151     <none>        9443/TCP            52d
webapp       ClusterIP   10.108.92.21    <none>        8000/TCP            52d
```
The role.yaml, role-binding.yaml and service-account.yaml permit the jenkins pod to make api calls with kubectl to the host kubernetes engine.
```
jcluser@ubuntu:~$ kubectl get role
NAME          CREATED AT
modify-pods   2023-08-22T13:17:17Z
```
```
jcluser@ubuntu:~$ kubectl get sa
NAME                   SECRETS   AGE
default                0         52d
internal-jenknis-pod   0         52d
```
Other special considerations are configmaps for jenkins and proxy:
```
kubectl create cm jenkins-keystore --from-file=/home/jcluser/nita-jenkins/certificates/jenkins_keystore.jks
kubectl create cm proxy-config-cm --from-file=/home/jcluser/nginx/nginx.conf
kubectl create cm proxy-cert-cm --from-file=/home/jcluser/nginx/certificates/
```
```
jcluser@ubuntu:~$ kubectl get cm
NAME               DATA   AGE
jenkins-keystore   1      52d
proxy-cert-cm      2      52d
proxy-config-cm    1      52d
```
