# Deploy microservices on K8S

## Prerequisites

- You have **Docker** installed
- You have a DockerHUB user
- You have a Minikube cluster up and running with `kubectl` CLI already configured to access it
- You setup all the infrastructure components as showed in lab [06 - Install infrastructure components on K8S](labs/06-Install_infrastructure_components_on_K8S/README.md)

## Deploy Customser microservice

Build the image (change the placeholder \<YOUR_DOCKERHUB_USER\> with your DockerHUB username)

```console
$ docker build --build-arg MVN_ARGS=-DskipTests  -t <YOUR_DOCKERHUB_USER>/customer:1.0-SNAPSHOT ../02-eCommerce_microservices/customer/
...
Successfully built af34dc2c05d5
Successfully tagged dennydgl1/customer:1.0-SNAPSHOT
```

Authenticate with DockerHUB (insert your credentials if prompted)

``` console
$ docker login
Authenticating with existing credentials...
Login Succeeded
```

Push the image on DockerHUB (change the placeholder \<YOUR_DOCKERHUB_USER\> with your DockerHUB username)

``` console
$ docker push <YOUR_DOCKERHUB_USER>/customer:1.0-SNAPSHOT
...
1.0-SNAPSHOT: digest: sha256:b51a9a0f4a1e640d5e91bce08a6b0a8e40403d704fead46fc83043a40ded0db0 size: 1366
```

Create the ConfigMap object to store **application.properties** file

```console
$ kubectl create configmap --from-file ../02-eCommerce_microservices/customer/src/main/resources/application.properties customer-conf
configmap/customer-conf created
```

Modify the image name in **customer-ms.yaml** file, in order to reflect the one you used in the previous steps (change the placeholder \<YOUR_DOCKERHUB_USER\> with your DockerHUB username)

```yaml
...
 image: <YOUR_DOCKERHUB_USER>/customer:1.0-SNAPSHOT
...
```

Apply the manifest

```console
$ kubectl apply -f customer-ms.yaml
deployment.apps/customer-deployment created
service/customer-service created
```

Check if everything went fine

```console
$ kubectl get deploy,pod,svc -l app=customer
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/customer-deployment   1/1     1            1           23s

NAME                                       READY   STATUS    RESTARTS   AGE
pod/customer-deployment-6df99bcbd4-dwgpr   1/1     Running   0          23s

NAME                       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/customer-service   NodePort   10.108.241.163   <none>        8102:32102/TCP   23s
```

Invoke the microservice:

```console
$ curl localhost:32102/customers-service/v2/customers/
[]
```

You should get an empty array just because at this stage the database is empty.

## Deploy Order microservice

## Deploy Notification microservice