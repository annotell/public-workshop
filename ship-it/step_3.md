# Deploying the Code
Next step will be to instead of running the docker container on you local computer, see how it would look when running it in Kubernetes. 
Usually Kubernetes is run in the cloud and over multiple instances, but for the sake of this workshop we'll use minikube which similates a Kubernetes cluster on your local computer. 

1. First step is to start minikube
  ```bash
  $ minikube start
  ```
  This might take a couple of minutes and the output will be:
  ```bash
  ➜  ~ minikube start
  😄  minikube v1.15.1 on Darwin 10.15.7
  ✨  Using the docker driver based on existing profile
  👍  Starting control plane node minikube in cluster minikube
  🔄  Restarting existing docker container for "minikube" ...
  🐳  Preparing Kubernetes v1.19.4 on Docker 19.03.13 ...
  🔎  Verifying Kubernetes components...
  🌟  Enabled addons: storage-provisioner, default-storageclass
  🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
  ```

  Minikube docker has it's own context, we need to rebuild our docker image (or pull for a remote repository(which we don't have now)) in the minikube context! So let's start with that:

2. Set up the minikube docker context:
  ```bash
  $ eval $(minikube docker-env)
  ```

3. Our next step is to rebuild the docker image, but let's pick a new tag:
  ```bash
  $ docker build -t datatjej:2.0 .
  ```
4. Make sure the image exists in the docker, and check that the tag is correct (2.0 in this case). Is there many images, a good thing is then to filter on a keyword. This you do with `| grep <KEYWORD>`. 
  ```bash
  $ docker images | grep datatjej
  ```

5. Make sure the datatjej-deployment.yaml points to the right image (the tag is the correct number)
```yaml
      containers:
      - name: datatjej
        image: datatjej:2.0
```

6. You have now made sure the image in the .yaml is correct, then it is time to   apply this to Kubernetes, but first we want to check if everything is ok
  ```bash
  $ kubectl apply -f datatjej-deployment.yaml --dry-run=client
  ```
  Expected output: 
  ```bash
  deployment.apps/datatjej created (dry run)
  ```
  If the output was as expected, we can run the same command again, but without dry-run and apply it for real:
  ```bash
  $ kubectl apply -f datatjej-deployment.yaml
  ```
  Expected output and the deployment is now applied to Kubernetes 
  ```bash
  deployment.apps/datatjej created
  ```

7. To check what deployments you have running in Kubernetes run:
  ```bash
  $ kubectl get deployments
  ```
  Our expected output is shown below where you can see the datatjej deployment
  ```bash
  NAME       READY   UP-TO-DATE   AVAILABLE   AGE
  datatjej   2/2     2            2           27s
  ```

8. Each pod maps one "docker run" on your laptop
  ```bash
  $ kubectl get pods
  ```
  The output tells you a bit of information about the pods. If something has gone wrong with any pod, you may get some information about that here.
  ```bash 
  NAME                        READY   STATUS    RESTARTS   AGE
  datatjej-5564cc49c5-h5n6d   1/1     Running   0          60s
  datatjej-5564cc49c5-t5zg4   1/1     Running   0          60s
```
9. The next step is to expose our application with a service
```bash
$ kubectl apply -f datatjej-service.yaml --dry-run=client
```
Expected output:
```bash
service/datatjej configured (dry run)
```
```bash
$ kubectl apply -f datatjej-service.yaml
```
```bash
service/datatjej created
```
```bash
$ kubectl get service datatjej
```

*** We can see that the service has two endpoints, mapping to our two pods
```bash
$ kubectl get endpoints datatjej
```
```bash
NAME       ENDPOINTS                         AGE
datatjej   172.17.0.3:8080,172.17.0.4:8080   4h28m
```
```bash
kubectl get pods -o wide
```
```bash
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
datatjej-7d74b7c88c-k2sx5   1/1     Running   0          65m   172.17.0.3   minikube   <none>           <none>
datatjej-7d74b7c88c-vglrc   1/1     Running   0          39s   172.17.0.4   minikube   <none>           <none>
```
**** compare output of pods with list of endpoints, they should match

We now have a service, and it's backed by two different pods! This allows us to have a higher available application since we can lose one of the pods and still servce traffic. But by just setting up a service like this, we're not exposing our application outside the minikube kubernetes cluster context, so in order to access the website, we need to set up a link to the service via minikube, once that is done, we can now view our website via the URL minikube returns, open the link in the browser
```bash
$ minikube service --url datatjej
```
```bash
😿  service default/datatjej has no node port
🏃  Starting tunnel for service datatjej.
|-----------|----------|-------------|------------------------|
| NAMESPACE |   NAME   | TARGET PORT |          URL           |
|-----------|----------|-------------|------------------------|
| default   | datatjej |             | http://127.0.0.1:63093 |
|-----------|----------|-------------|------------------------|
http://127.0.0.1:63093
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.

ctrl+c to exit the tunnel
```

Return to the [workshop guidelines](./README.md)