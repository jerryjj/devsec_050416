# Demo of K8s Secrets

## Demo 1: Using kubectl create secret

Lets assume we have two files in our filesystem whose contents we want to expose
as secrets.

Lets create those files first:

```sh
mkdir -p ./tmp/manual
echo "admin" > ./tmp/manual/username.txt
echo "really-secret-password" > ./tmp/manual/password.txt
```

Now we can utilize the _kubectl create secret_ command to turn these files to a
single Secret and deploy it to the K8s cluster.

```sh
kubectl create secret generic db-user-pass \
--from-file=./tmp/manual/username.txt \
--from-file=./tmp/manual/password.txt
```

Lets check that the secret was created properly:

```sh
kubectl get secrets
kubectl describe secrets/db-user-pass
```

Note that neither _get_ nor _describe_ shows the contents of the file by default.
This is to protect the secret from being exposed accidentally
to someone looking or from being stored in a terminal log.

To decode the values we have in our secrets we can run the following commands:

```sh
kubectl get secret db-user-pass -o yaml
echo "cmVhbGx5LXNlY3JldC1wYXNzd29yZAo=" | base64 -D
```

Lets cleanup our steps for now:

```sh
kubectl delete secret db-user-pass
rm -fR ./tmp/manual
```

## Demo 2: Using kubectl create and using Secrets in a Pod

Next lets create a secret manually with the Secret-object type.
First lets create a Secret definition file to path _./tmp/mysecret.yaml_:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: cmVhbGx5LXNlY3JldC1wYXNzd29yZAo=
  username: YWRtaW4K
```

The contents to the _data_ has been generated as such:

```sh
echo "admin" | base64
echo "really-secret-password" | base64
```

Now lets deploy this secret using the _kubectl_ -command.

```sh
kubectl create -f ./tmp/mysecret.yaml
```

Again lets make sure we can see our Secret:

```sh
kubectl get secrets
```

Now lets create a simple Pod that binds our "mysecret" secrets to environment
and echo them to console.

```sh
kubectl create -f ./mysecret-env-pod.yaml
kubectl logs mysecret-env-pod | grep "SECRET_"
```

Lets remove that Pod and create one that mounts the Secrets to Filesystem
and echo the contents of the username-file which we mount to /etc/mysecrets/username.

```sh
kubectl delete pods mysecret-env-pod
kubectl create -f ./mysecret-fs-pod.yaml
kubectl logs mysecret-fs-pod
```

And lets cleanup:

```sh
kubectl delete pods mysecret-fs-pod
rm ./tmp/mysecret.yaml
```
