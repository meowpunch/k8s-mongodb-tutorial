# k8s mongodb tutorial
This demo was implemented, just following [Nana k8s tutorial](https://www.youtube.com/watch?v=X48VuDVv0do)

- deploy two applications, mongodb(database) and mongo express(web application), on kubernetes

## Architecture Overview
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) / [Pods](https://kubernetes.io/docs/concepts/workloads/pods/): containerized applications are running on pods.
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/): expose an application on a set of pods as a network service.
    - mongodb - internal service
    - mongo express - external service
- [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/): storing configuration variables e.g. db url
- [Secret](https://kubernetes.io/docs/concepts/configuration/secret/): storing secret env variables e.g. db username and pwd

## Environment
```shell
$ minikube version 
minikube version: v1.24.0

$ kubectl version 
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.3", GitCommit:"ca643a4d1f7bfe34773c74f79527be4afd95bf39", GitTreeState:"clean", BuildDate:"2021-07-15T21:04:39Z", GoVersion:"go1.16.6", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.3", GitCommit:"c92036820499fedefec0f847e2054d824aea6cd1", GitTreeState:"clean", BuildDate:"2021-10-27T18:35:25Z", GoVersion:"go1.16.9", Compiler:"gc", Platform:"linux/amd64"}
```

## Get Started
- check if there are existing services other than kubernetes cluster
```shell
kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   53d
```

- deploy mongodb secret, root username and pwd
```shell
$ kubectl apply -f mongo-secret.yml
secret/mongodb-secret created

$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-8tgsg   kubernetes.io/service-account-token   3      53d
mongodb-secret        Opaque                                2      10s
```

- deploy mongodb-deployment
```shell
$ kubectl apply -f mongo.yml       
deployment.apps/mongodb-deployment created
service/mongodb-service created

$ kubectl get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/mongodb-deployment-8f6675bc5-6hpzx   1/1     Running   0          20s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP    53d
service/mongodb-service   ClusterIP   10.109.234.129   <none>        3000/TCP   20s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb-deployment   1/1     1            1           20s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-deployment-8f6675bc5   1         1         1       20s
```

- deploy mongo configmap, database url
```shell
$ kubectl apply -f mongo-configmap.yml
configmap/mongodb-configmap created

$ kubectl get configmap               
NAME                DATA   AGE
kube-root-ca.crt    1      53d
mongodb-configmap   1      22s
```

- deploy mongo express
```shell
$  kubectl apply -f mongo-express.yml  
deployment.apps/mongo-express created
service/mongo-express-service created

$ kubectl get pod                   
NAME                                 READY   STATUS    RESTARTS   AGE
mongo-express-74d886864-9n9n9        1/1     Running   0          8s
mongodb-deployment-8f6675bc5-6hpzx   1/1     Running   0          2m36s

$ kubectl logs mongo-express-74d886864-9n9n9
Welcome to mongo-express
------------------------


(node:7) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
Mongo Express server listening at http://0.0.0.0:8081
Server is open to allow connections from anyone (0.0.0.0)
basicAuth credentials are "admin:pass", it is recommended you change this in your config.js!

```

- pending external ip adress to mongo-express-service
```shell
$  kubectl get service                       
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          53d
mongo-express-service   LoadBalancer   10.98.234.125    <pending>     8080:30000/TCP   2m12s
mongodb-service         ClusterIP      10.109.234.129   <none>        3000/TCP         4m40s

$ minikube service mongo-express-service
|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | mongo-express-service |        8080 | http://192.168.49.2:30000 |
|-----------|-----------------------|-------------|---------------------------|
🏃  Starting tunnel for service mongo-express-service.
|-----------|-----------------------|-------------|------------------------|
| NAMESPACE |         NAME          | TARGET PORT |          URL           |
|-----------|-----------------------|-------------|------------------------|
| default   | mongo-express-service |             | http://127.0.0.1:53043 |
|-----------|-----------------------|-------------|------------------------|
🎉  Opening service default/mongo-express-service in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.

```

## Implementation
If you check it out [mongodb official image](https://hub.docker.com/_/mongo) on dockerhub
- standard mongodb port `27017`
- you can adjust environment variables `MONGO_INITDB_ROOT_IUSERNAME` and `MONGO_INITDB_ROOT_PASSWORD`
  - I will manage root username and pwd as Secret

Same as mongo express [official image](https://hub.docker.com/_/mongo-express)

### Secret
 
- with [Basic authentication Secret](https://kubernetes.io/docs/concepts/configuration/secret/#basic-authentication-secret)
```shell
$ echo -n 'username' | base64
dXNlcm5hbWU=

$ echo -n 'password' | base64
cGFzc3dvcmQ=
```

- configure secret
```yaml
# mongo-secret.yml
apiVersion: v1
kind: Secret
metadata:
    name: mongodb-secret
type: Opaque
data:
    mongo-root-username: dXNlcm5hbWU=
    mongo-root-password: cGFzc3dvcmQ=
```

- consume env variables
```yaml
# mongo.yml
...
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
...
```
