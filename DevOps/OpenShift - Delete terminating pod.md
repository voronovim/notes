# OpenShift - Delete terminating pod

Step 1: Delete pod forcefully

```sh
oc get pods
```

```sh 
oc delete pod $POD_NAME -n myproject --grace-period=0 --force
```

Step 2: Remove deletionTimestamp
```sh
oc edit pod $POD_NAME
```
Before:
`deletionTimestamp: 2019-01-23T11:40:28Z`

After
`deletionTimestamp: null`

Step 3: Remove items under metadata.finalizers

Before:
```yaml
...
metadata:
   finalizers:
   - foregroundDeletion 
...
```

After:
```yaml
...
metadata:
   finalizers: null
...
```

Check our pods again.

```sh
oc get pods
```

If not working:

Step 4: Invoke OpenShift API

This is a bit risky method as incorrect usage of this API method may result in unpredictable situations in your OpenShift cluster environment; so be careful.

This method is explained in Red Hat KnowledgeBase with sample instruction set. Let me give some samples below.

We can invoke OpenShift API directly for object manipulation, even for deletion in our case. To delete a pod stuck in ‘Terminating‘ or ‘Unknown‘ state, you may try following curl sent to the API:

```sh
echo '{ "propagationPolicy": "Background" }' | curl -k -X DELETE -d @- -H "Authorization: Bearer XYZ" -H 'Accept: application/json' -H 'Content-Type: application/json'  https://master.example.com:443/api/v1/namespaces/myproject/pods/my-app-123
```

Where:

```properties
master_URL = master.example.com
port = 443
token = XYZ (this one you have to get using oc whoami -t command)
pod_name = my-app-123
namespace = myproject
```



