# Performance benchmark operator: Installation and how to use it.

# Install Helm on root of cash node
1. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
2. chmod 700 get_helm.sh
3. ./get_helm.sh

Downloading https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

4. helm version
version.BuildInfo{Version:"v3.6.3", GitCommit:"d506314abfb5d21419df8c7e7e68012379db2354", GitTreeState:"clean", GoVersion:"go1.16.5"}

# Now to run benchmark operator we create new project and install operator in it using helm charts 
``` ##################################################################################################################### ``` 
# Create new project named "ripsaw" 
[core@r189 ~]$ oc new-project my-ripsaw
Now using project "my-ripsaw" on server "https://api.r189a.dsp.lab:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname

``` ##################################################################################################################### ``` 
# Create admin policy for the new project 
[core@r189 ~]$ oc adm policy -n my-ripsaw add-scc-to-user privileged -z benchmark-operator
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "benchmark-operator"

``` ##################################################################################################################### ``` 
# Clone the bull-dozer/benchmark-operator (master branch)
[core@r189 ~]$ git clone https://github.com/cloud-bulldozer/benchmark-operator

Cloning into 'benchmark-operator'...
remote: Enumerating objects: 6490, done.
remote: Counting objects: 100% (904/904), done.
remote: Compressing objects: 100% (450/450), done.
remote: Total 6490 (delta 459), reused 718 (delta 375), pack-reused 5586
Receiving objects: 100% (6490/6490), 9.36 MiB | 22.62 MiB/s, done.
Resolving deltas: 100% (4170/4170), done.


[core@r189 ~]$ cd benchmark-operator/

``` ##################################################################################################################### ``` 

[core@r189 benchmark-operator]$ pwd
/home/core/benchmark-operator

# Run make deploy to apply its changes 
[core@r189 benchmark-operator]$ make deploy
``` ##################################################################################################################### ``` 
# Navigate to the charts directory 
[core@r189 benchmark-operator]$ cd charts/benchmark-operator/
[core@r189 benchmark-operator]$ pwd
/home/core/benchmark-operator/charts/benchmark-operator

``` ##################################################################################################################### ``` 
# Install operator using helm chart
[core@r189 benchmark-operator]$ helm install benchmark-operator . -n my-ripsaw --create-namespace
walk.go:74: found symbolic link in path: /home/core/benchmark-operator/charts/benchmark-operator/crds/crd.yaml resolves to /home/core/benchmark-operator/config/crd/bases/ripsaw.cloudbulldozer.io_benchmarks.yaml
NAME: benchmark-operator
LAST DEPLOYED: Tue Aug  3 05:37:34 2021
NAMESPACE: my-ripsaw
STATUS: deployed
REVISION: 1
TEST SUITE: None

``` ##################################################################################################################### ``` 
# Verify operator is installed in given namespace using 'helm list'
[core@r189 benchmark-operator]$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                         APP VERSION
benchmark-operator      my-ripsaw       1               2021-08-03 05:37:34.604945612 -0400 EDT deployed        benchmark-operator-0.1.0      1.16.0


``` ##################################################################################################################### ``` 
[core@r189 benchmark-operator]$ oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
dpdk     rendered-dpdk-286ff8ab0d64ec916c4fbe536565ae9f     True      False      False      1              1                   1                     0                      18d
master   rendered-master-5750e542ee71f07b7a47d9744ba499f4   True      False      False      3              3                   3                     0                      32d
worker   rendered-worker-479f118a77224975faf9bb3afb44d481   True      False      False      2              2                   2      

``` ##################################################################################################################### ``` 
# Label machine config pool
[core@r189 benchmark-operator]$ oc label mcp worker machineconfiguration.openshift.io/role=worker
machineconfigpool.machineconfiguration.openshift.io/worker labeled

``` ##################################################################################################################### ``` 
# Verify that label is attached
[core@r189 benchmark-operator]$ oc get mcp --show-labels
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE   LABELS
dpdk     rendered-dpdk-286ff8ab0d64ec916c4fbe536565ae9f     True      False      False      1              1                   1                     0                      18d   machineconfiguration.openshift.io/role=worker-cnf
master   rendered-master-5750e542ee71f07b7a47d9744ba499f4   True      False      False      3              3                   3                     0                      32d   machineconfiguration.openshift.io/mco-built-in=,operator.machineconfiguration.openshift.io/required-for-upgrade=,pools.operator.machineconfiguration.openshift.io/master=
worker   rendered-worker-479f118a77224975faf9bb3afb44d481   True      False      False      2              2                   2                     0                      32d   machineconfiguration.openshift.io/mco-built-in=,machineconfiguration.openshift.io/role=worker,pools.operator.machineconfiguration.openshift.io/worker=

``` ##################################################################################################################### ``` 
# Create peroformance profile (Note: above step is necessary before creating the performance profile)
[core@r189 benchmark-operator]$ oc get performanceprofiles
NAME                      AGE
r640-performanceprofile   18d

``` ##################################################################################################################### ``` 
# check 
[core@r189 benchmark-operator]$ oc get sriovnetworknodestate -A
NAMESPACE                          NAME           AGE
openshift-sriov-network-operator   cmp1.dsp.lab   24d
openshift-sriov-network-operator   cmp2.dsp.lab   24d
openshift-sriov-network-operator   cmp3.dsp.lab   24d

``` ##################################################################################################################### ``` 
# Checklist:
1. Install SRIOV and performance addon operator
2. Install SRIOV network node policy 
3. Install SRIOV network 
4. Enable DPDK, hugepages, add labels to worker which will be used as dpdk worker nodes 
``` ##################################################################################################################### ``` 
# Here we have compute 3 
[core@r189 benchmark-operator]$ oc get nodes
NAME           STATUS   ROLES               AGE   VERSION
cc1.dsp.lab    Ready    master              32d   v1.19.0+b00ba52
cc2.dsp.lab    Ready    master              32d   v1.19.0+b00ba52
cc3.dsp.lab    Ready    master              32d   v1.19.0+b00ba52
cmp1.dsp.lab   Ready    worker              32d   v1.19.0+b00ba52
cmp2.dsp.lab   Ready    worker              32d   v1.19.0+b00ba52
cmp3.dsp.lab   Ready    worker,worker-cnf   32d   v1.19.0+b00ba52

``` ##################################################################################################################### ``` 
# Verify hugepages are enabled on worker node
[core@r189 benchmark-operator]$ ssh cmp3.dsp.lab cat /proc/meminfo | grep -i huge
HugePages_Total:      16
HugePages_Free:       15
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB
Hugetlb:        16777216 kB

``` ##################################################################################################################### ``` 
1. create mlx network in ripsaw ns 
2. create benchmark manifest 

``` ##################################################################################################################### ``` 
# Create MCP 
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: worker-cnf
  labels:
    machineconfiguration.openshift.io/role: worker-cnf
spec:
  machineConfigSelector:
    matchExpressions:
      - {
          key: machineconfiguration.openshift.io/role,
          operator: In,
          values: [worker-cnf, worker],
        }
  paused: false
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/worker-cnf: ""

``` ##################################################################################################################### ``` 
# Create perf operator 
# Running oc create -f pao-operator.yaml will create the namespace, OperatorGroup, and Subscription: needed for the PAO Operator installation

apiVersion: v1
kind: Namespace
metadata:
  name: openshift-performance-addon-operator
  labels:
    openshift.io/run-level: "1"
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-performance-addon-operator
  namespace: openshift-performance-addon-operator
spec:
  targetNamespaces:
  - openshift-performance-addon-operator
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-performance-addon-operator-subscription
  namespace: openshift-performance-addon-operator
spec:
  channel: "4.6" 
  name: performance-addon-operator
  source: redhat-operators 
  sourceNamespace: openshift-marketplace


``` ##################################################################################################################### ``` 
# Create perf profile
# PerformanceProfile for real time kernel
# Using SMT
# Hardware Used: PowerEdge XE2420 Servers
apiVersion: performance.openshift.io/v1
kind: PerformanceProfile
metadata:
  name: 640-performanceprofile
spec:
  additionalKernelArgs:
    - nmi_watchdog=0
    - audit=0
    - mce=off
    - processor.max_cstate=1
    - idle=poll
    - intel_idle.max_cstate=0
  cpu:
    isolated: "2-39,42-79"
    reserved: "0,1,40,41,"
  hugepages:
    defaultHugepagesSize: 1G
    pages:
    - size: "1G"
      count: 16
  realTimeKernel:
    enabled: true
  numa:
    topologyPolicy: "restricted"
    # topologyPolicy: "best-effort" # May change performance, but this can be used
  nodeSelector:
    node-role.kubernetes.io/worker-cnf: ""

``` ##################################################################################################################### ``` 

# Create MLX node policy 
[core@r189 perf-operator-test]$ vim mlx-node-policy.yaml
[core@r189 perf-operator-test]$ cat mlx-node-policy.yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: mellanox-dpdk-node-policy
  namespace: openshift-sriov-network-operator
spec:
  resourceName: mellanoxnics
  nodeSelector:
    node-role.kubernetes.io/worker-cnf: ""
  numVfs: 5
  nicSelector:
    pfNames: ["ens3f0"]
    rootDevices: ["0000:d8:00.0"]
  deviceType: netdevice
  isRdma: true

``` ##################################################################################################################### ``` 
# Create sriov network
[core@r189 perf-operator-test]$ vim testpd-seriov-nw.yaml
[core@r189 perf-operator-test]$ cat testpd-seriov-nw.yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetwork
metadata:
  name: testpmd-sriov-network
  namespace: openshift-sriov-network-operator
spec:
  ipam: |
    {
      "type": "host-local",
      "subnet": "10.57.1.0/24",
      "rangeStart": "10.57.1.100",
      "rangeEnd": "10.57.1.200",
      "routes": [{
        "dst": "0.0.0.0/0"
      }],
      "gateway": "10.57.1.1"
    }
  spoofChk: "on"
  trust: "on"
  resourceName:  mellanoxnics
  networkNamespace: my-ripsaw


# Check network is created.
[core@r189 perf-operator-test]$ oc create -f testpd-seriov-nw.yaml
sriovnetwork.sriovnetwork.openshift.io/testpmd-sriov-network created

[core@r189 perf-operator-test]$ oc get sriovnetworks -n openshift-sriov-network-operator
NAME                      AGE
cmp1-sriov-network        28h
cmp2-sriov-network-170    28h
cmp3-dpdk-network         19d
mellanox-dpdk-network     18d
mellanox-dpdk-network-1   17d
> testpmd-sriov-network     20s

``` ##################################################################################################################### ``` 
# Running TestPMD and TRex pods
[core@r189 perf-operator-test]$ vim dpdktest.yaml
[core@r189 perf-operator-test]$ cat dpdktest.yaml
apiVersion: ripsaw.cloudbulldozer.io/v1alpha1
kind: Benchmark
metadata:
  name: testpmd-benchmark
  namespace: my-ripsaw
spec:
  clustername: r189a
  workload:
    name: testpmd
    args:
      privileged: true
      pin: false
      pin_testpmd: "cmp3.dsp.lab"
      pin_trex: "cmp3.dsp.lab"
      networks:
        testpmd:
          - name: testpmd-sriov-network
            count: 2  # Interface count, Min 2
        trex:
          - name: testpmd-sriov-network
            count: 2

# Run command to create it
[core@r189 perf-operator-test]$ oc create -f dpdktest.yaml
benchmark.ripsaw.cloudbulldozer.io/testpmd-benchmark created

# Verify 
[core@r189 perf-operator-test]$ oc get benchmark
NAME                TYPE      STATE      METADATA STATE   CERBERUS        SYSTEM METRICS   UUID                                   AGE
testpmd-benchmark   testpmd   Starting   not collected    not connected   Not collected    d2dc4f05-1978-594f-a9fd-2a6b028997ef   58s
``` ##################################################################################################################### ``` 

# State is starting..... because role bindind and service account is mapped to benchmark-operator 
[core@r189 perf-operator-test]$ oc get rolebinding
NAME                              ROLE                                          AGE
admin                             ClusterRole/admin                             119m
benchmark-operator                Role/benchmark-operator                       107m
system:deployers                  ClusterRole/system:deployer                   119m
system:image-builders             ClusterRole/system:image-builder              119m
system:image-pullers              ClusterRole/system:image-puller               119m
system:openshift:scc:privileged   ClusterRole/system:openshift:scc:privileged   118m


[core@r189 perf-operator-test]$
[core@r189 perf-operator-test]$
[core@r189 perf-operator-test]$
[core@r189 perf-operator-test]$ oc get rolebinding benchmark-operator
NAME                 ROLE                      AGE
benchmark-operator   Role/benchmark-operator   108m

# serviceaccount "benchmark-operator" is associated with a role "benchmark-operaror" as given below
[core@r189 perf-operator-test]$ oc get rolebinding benchmark-operator -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  annotations:
    meta.helm.sh/release-name: benchmark-operator
    meta.helm.sh/release-namespace: my-ripsaw
  creationTimestamp: "2021-08-03T09:37:34Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:meta.helm.sh/release-name: {}
          f:meta.helm.sh/release-namespace: {}
        f:labels:
          .: {}
          f:app.kubernetes.io/managed-by: {}
      f:roleRef:
        f:apiGroup: {}
        f:kind: {}
        f:name: {}
      f:subjects: {}
    manager: helm
    operation: Update
    time: "2021-08-03T09:37:34Z"
  name: benchmark-operator
  namespace: my-ripsaw
  resourceVersion: "14474137"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/my-ripsaw/rolebindings/benchmark-operator
  uid: 28f86edc-f7c1-4659-9439-e7c464cdcf15
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: benchmark-operator
subjects:
- kind: ServiceAccount
  name: benchmark-operator
  namespace: my-ripsaw


``` ##################################################################################################################### ``` 
# Verify what all permissions the role benchmark-operator has...

[core@r189 perf-operator-test]$ oc get role
NAME                 CREATED AT
benchmark-operator   2021-08-03T09:37:34Z


[core@r189 perf-operator-test]$ oc get role benchmark-operator -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    meta.helm.sh/release-name: benchmark-operator
    meta.helm.sh/release-namespace: my-ripsaw
  creationTimestamp: "2021-08-03T09:37:34Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:meta.helm.sh/release-name: {}
          f:meta.helm.sh/release-namespace: {}
        f:labels:
          .: {}
          f:app.kubernetes.io/managed-by: {}
      f:rules: {}
    manager: helm
    operation: Update
    time: "2021-08-03T09:37:34Z"
  name: benchmark-operator
  namespace: my-ripsaw
  resourceVersion: "14474135"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/my-ripsaw/roles/benchmark-operator
  uid: e47beeab-3546-4d94-ab7c-34772b8243e8
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - daemonsets
  - services
  - endpoints
  - persistentvolumeclaims
  - virtualmachineinstances
  - events
  - configmaps
  - secrets
  - pods/log
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  - deployments/finalizers
  verbs:
  - '*'
- apiGroups:
  - monitoring.coreos.com
  resources:
  - servicemonitors
  verbs:
  - get
  - create
- apiGroups:
  - kubevirt.io
  resources:
  - virtualmachineinstances
  verbs:
  - '*'
- apiGroups:
  - ripsaw.cloudbulldozer.io
  resources:
  - '*'
  verbs:
  - '*'
- apiGroups:
  - batch
  - extensions
  resources:
  - jobs
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete
- apiGroups:
  - policy
  resourceNames:
  - privileged
  resources:
  - podsecuritypolicies
  verbs:
  - use
- apiGroups:
  - security.openshift.io
  resourceNames:
  - privileged
  resources:
  - securitycontextconstraints
  verbs:
  - use
- apiGroups:
  - hyperfoil.io
  resources:
  - hyperfoils
  verbs:
  - '*'
- apiGroups:
  - networking.k8s.io
  resources:
  - networkpolicies
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete

``` ##################################################################################################################### ``` 
# Edit role and add below mentioned lines at the end. 
[core@r189 perf-operator-test]$ oc edit role benchmark-operator
role.rbac.authorization.k8s.io/benchmark-operator edited

> we need to add APIgroup 'k8s.cni.cncf.io' with resources 'network-attachment-definitions' and verbs '*'
>>>
- apiGroups:
  - k8s.cni.cncf.io
  resources:
  - network-attachment-definitions
  verbs:
  - '*'

``` ##################################################################################################################### ``` 

``` ##################################################################################################################### ``` 

``` ##################################################################################################################### ``` 

``` ##################################################################################################################### ``` 
