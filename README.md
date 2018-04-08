# build

```
./build.sh
```

# set credential

```
//set user1
$ pi config set-credentials user1 --region=gcp-us-central1 --access-key="xxx" --secret-key="xxxxxx"

//set user2
$ pi config set-credentials user2 --region=gcp-us-central1 --access-key="xxx" --secret-key="xxxxxx"
```

# use credential

```
$ pi --user=user1 get nodes
or
$ pi --user=user2 get nodes
or
$ pi --access-key=xxx --secret-key=xxxxxx get nodes
or
$ pi --server=https://gcp-us-central1.hypersh --access-key=xxx --secret-key=xxxxxx get nodes
or
$ pi --region=gcp-us-central1 --access-key=xxx --secret-key=xxxxxx get nodes
```

# config

**priority**:  command line arguments > config file parameters


```
// argument:
--host
--region
--access-key
--secret-key


// config file:
$ cat ~/.pi/config
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://*.hyper.sh:6443
  name: default
contexts:
- context:
    cluster: default
    namespace: default
    user: testuser3
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: testuser2
  user:
    access-key: xxx
    region: gcp-us-central1
    secret-key: xxxxxx
- name: testuser3
  user:
    access-key: xxx
    region: gcp-us-central1
    secret-key: xxxxxx
```

# usage

## nodes operation example

```
$ pi get nodes
NAME              STATUS    ROLES     AGE       VERSION
gcp-us-central1   Ready     <none>    4d

$ pi get nodes --show-labels
NAME              STATUS    ROLES     AGE       VERSION   LABELS
gcp-us-central1   Ready     <none>    6d                  availabilityZone=gcp-us-central1-b|UP,defaultZone=gcp-us-central1-b,email=testuser3@test.com,resources=pod:3/20,volume:6/40,fip:0/5,service:1/5,secret:1/3,serviceClusterIPRange=10.96.0.0/12,tenantID=00a54ebcc0444bb384e48f6fd7b5597b
```

## pod operation example

```
// create pod via yaml
$ pi create -f examples/pod/pod-nginx.yaml
pod/nginx


// list pods
$ pi get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          12s


// get pod
$ pi get pods nginx -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      id: a0eabb46fccd616f4a22cbeec8547c344c3a85b7a1b95c1f88c17bb4d27b1267
      sh_hyper_instancetype: s4
      zone: gcp-us-central1-b
    creationTimestamp: 2018-04-08T14:39:52Z
    labels:
      app: nginx
    name: nginx
    namespace: default
    uid: b2aa75df-3b3a-11e8-b8a4-42010a000032
  spec:
    containers:
    - image: nginx
      name: nginx
      resources: {}
    nodeName: gcp-us-central1
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: 2018-04-08T14:39:52Z
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: 2018-04-08T14:39:56Z
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: 2018-04-08T14:39:52Z
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: hyper://2a077fec7051be72cb9435a7114c66c0d43cf66e7f6667cb34b28eeeea374a54
      image: sha256:c5c4e8fa2cf7d87545ed017b60a4b71e047e26c4ebc71eb1709d9e5289f9176f
      imageID: sha256:c5c4e8fa2cf7d87545ed017b60a4b71e047e26c4ebc71eb1709d9e5289f9176f
      lastState: {}
      name: nginx
      ready: true
      restartCount: 0
      state:
        running:
          startedAt: 2018-04-08T14:39:56Z
    phase: Running
    podIP: 10.244.58.203
    qosClass: Burstable
    startTime: 2018-04-08T14:39:52Z
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""


// delete pod
$ pi delete pod nginx --now
pod "nginx" deleted
```

## service operation example

```
// create service via yaml
$ pi create -f examples/service/service-clusterip-default.yaml
service/test-clusterip-default


// list services
$ pi get service
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
test-clusterip-default   ClusterIP   10.105.34.132   <none>        8080/     15s


// get service
$ pi get service clusterip -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    id: 60de055bc7027e49bcad66fa336e6313e3acd322a0836caecda039610eaf20bd
  creationTimestamp: 2018-04-08T14:43:37Z
  name: test-clusterip-default
  namespace: default
  uid: 3871f7a5-3b3b-11e8-b8a4-42010a000032
spec:
  clusterIP: 10.105.34.132
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
status:
  loadBalancer: {}


// delete service
$ pi delete services test-clusterip-default
service "test-clusterip-default" deleted
```

## secret operation example

```
// create secret via yaml
$ pi create -f examples/secret/secret-dockerconfigjson.yaml
secret "test-secret-dockerconfigjson" created


// create secret via command line arguments
$ pi create secret docker-registry test-secret-gitlab --docker-server=https://registry.gitlab.com --docker-username=xjimmy --docker-password=xxx --docker-email=xjimmyshcn@gmail.com
secret "test-secret-gitlab" created


// list all secrets
$ pi get secrets
NAME                           TYPE                             DATA      AGE
test-secret-dockerconfigjson   kubernetes.io/dockerconfigjson   1         2m
test-secret-gitlab             kubernetes.io/dockerconfigjson   1         8s


// delete secret
$ pi delete secrets test-secret-gitlab
secret "test-secret-gitlab" deleted
```

## volume operation example

```
// create volume
$ pi create volume vol1 --size=1
volume vol1(1GB) created in zone gcp-us-central1-b

// list volumes
$ pi get volumes
NAME              ZONE               SIZE(GB)  CREATEDAT                  POD
test-performance  gcp-us-central1-b  50        2018-03-26T05:31:05+00:00  test-flexvolume

// get volume
$ pi get volumes test-performance -o json
[
  {
    "name": "test-performance",
    "size": 50,
    "zone": "gcp-us-central1-b",
    "pod": "test-flexvolume",
    "createdAt": "2018-03-26T05:31:05.773Z"
  }

// delete volume
$ pi delete volume vol1
volume vol1 deleted

```

# fip operation example

```
$ pi create fip
35.192.x.x

$ pi create fip --count=2
35.188.x.x
35.189.x.x


$ pi get fips
FIP             NAME  CREATEDAT
35.192.x.x            2018-04-08T15:27:49+00:00
35.188.x.x            2018-04-08T15:31:08+00:00

$ pi name fip 35.192.x.x --name=test
fip 35.192.x.x renamed to test


$ pi get fips 35.192.x.x -o json
{
  "fip": "35.192.x.x",
  "name": "test",
  "createdAt": "2018-04-08T15:27:49.862Z",
  "pods": []
}


$ pi delete fips 35.188.x.x
fip "35.188.x.x" deleted
```