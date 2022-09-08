# golang-k3s
Develop Build and Deploy Golang App to K3s

### Container Runtime
We will require a container runtime as we will be building a image from our application code :

``` bash
$ curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
$ docker version
```

Build the image using the Dockerfile and by using the image name and tag of your own account:

``` bash
$ docker build -t shrutianekar/rpi-hostname:latest .

Now the image has been built, let's test this locally:

``` bash
$ docker run -it -p 80:8000 shrutianekar/rpi-hostname:latest
Server listening on port 8000
```

Make a HTTP Request:

``` bash
$ curl -i http://192.168.0.100/
HTTP/1.1 200 OK
Date: Sat, 10 Sep 2022 13:14:20 GMT
Content-Length: 23
Content-Type: text/plain; charset=utf-8

Hostname: 7c5f0d0bc212
```

Push the image to your image repository of choice
``` bash
$ docker push ruanbekker/rpi-hostname
```

From the configuration above we are using Traefik for our ingress and mapping port 80 to the k3s-demo service via the dns name k3s-demo.example.org

As I do not own example.org, I will be setting that in my /etc/hosts file:

``` bash
$ cat /etc/hosts | grep k3s
192.168.0.100 k3s-demo.example.org
```

Deploy our application:

``` bash
$ kubectl apply -f k3s-demo.yml
deployment.extensions/k3s-demo created
service/k3s-demo created
ingress.extensions/k3s-demo created
```

After a minute or so, have a look at the services:

``` bash
$ kubectl get service
NAMESPACE     NAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
default       k3s-demo     ClusterIP      10.43.89.51    <none>          80/TCP                       31s
```

The deployments:

``` bash
$ kubectl get deployments
NAMESPACE     NAME       READY   UP-TO-DATE   AVAILABLE   AGE
default       k3s-demo   1/1     1            1           42s
```

And get the pods:

``` bash
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
k3s-demo-777b9b7799-68kxg   1/1     Running   0          49s
```

Testing our Application
Now that our application is deployed and our hosts file is set, let's test the application:

``` bash
$ curl k3s-demo.example.org
Hostname: k3s-demo-777b9b7799-68kxg
```

And as you can see the Traefik accepted the connection and routed to the correct container.
