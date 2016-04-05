# Demo of K8s ConfigMaps

## Demo 1: Using kubectl create configmap

Lets assume we have two files in our filesystem whose contents we want to expose
as ConfigMaps.

Lets create those files first:

```sh
mkdir -p ./tmp/configs
cat <<EOF > ./tmp/configs/backend.properties
flags.feature1=true
flags.feature2=false
EOF
cat <<EOF > ./tmp/configs/frontend.properties
cdn.enabled=true
bgColor=red
EOF
```

Now we can utilize the _kubectl create configmap_ command to turn these files to a
single ConfigMap and deploy it to the K8s cluster.

```sh
kubectl create configmap app-config --from-file=./tmp/configs
```

Lets check that the secret was created properly:

```sh
kubectl get configmaps
kubectl describe configmaps app-config
```

You can see the two keys in the map are created from the filenames in the directory we pointed kubectl to. Since the content of those keys may be large, in the output of kubectl describe, you’ll see only the names of the keys and their sizes.

To decode the values we have in our secrets we can run the following commands:

```sh
kubectl get configmaps app-config -o yaml
```

Lets cleanup our steps for now:

```sh
kubectl delete configmaps app-config
rm -fR ./tmp/configs
```

We can also create configmaps from literal values straight from the command-line.

```sh
kubectl create configmap literal-config --from-literal=special.how=very --from-literal=special.type=charm
kubectl get configmaps literal-config -o yaml
kubectl delete configmaps literal-config
```

## Demo 2: Using kubectl create and using ConfigMaps in a Pod

Next lets create a configmap manually with the ConfigMap-object type.
First lets create our config definition file to path _./tmp/myconfig.yaml_:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfig
  namespace: default
data:
  backend.properties: |
    flags.feature1=true
    flags.feature2=false
  config.json: |
    {"key": "value"}
  bg.color: red
```

Now lets deploy this config using the _kubectl_ -command.

```sh
kubectl create -f ./tmp/myconfig.yaml
```

Again lets make sure we can see our ConfigMap:

```sh
kubectl get configmaps myconfig -o yaml
```

Now lets create a simple Pod that binds our "myconfig" condfis to environment
and echo them to console.

```sh
kubectl create -f ./myconfig-env-pod.yaml
kubectl logs myconfig-env-pod | grep "CFG_"
```

Lets remove that Pod and create one that mounts the ConfigMap to Filesystem
and echo the contents of the backend-file which we mount to /etc/myconfig/backend.properties.

```sh
kubectl delete pods myconfig-env-pod
kubectl create -f ./myconfig-fs-pod.yaml
kubectl logs myconfig-fs-pod
```

And lets cleanup:

```sh
kubectl delete pods myconfig-fs-pod
rm ./tmp/myconfig.yaml
```
