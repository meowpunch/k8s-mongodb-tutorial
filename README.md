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
If you check it out [mongodb official image](https://hub.docker.com/_/mongo) on dockerhub
- standard mongodb port `27017`
- you can adjust environment variables `MONGO_INITDB_ROOT_IUSERNAME` and `MONGO_INITDB_ROOT_PASSWORD`
  - I will manage root username and pwd as Secret

### Secret
 
 - with [Basic authentication Secret](https://kubernetes.io/docs/concepts/configuration/secret/#basic-authentication-secret)
```shell
$ echo -n 'db username' | base64
ZGIgdXNlcm5hbWU=

$ echo -n 'db password' | base64
ZGIgcGFzc3dvcmQ=
```
```shell
$ kubectl apply -f mongodb-secret.yml 
secret/mongodb-secret created

$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-8tgsg   kubernetes.io/service-account-token   3      53d
mongodb-secret        Opaque                                2      10s
```

