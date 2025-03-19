<pre>
<h1>Cluster Self-service Setup   </h1>

Configure a cluster with default settings for self-service projects.
Outcomes
• Create a project template that sets quotas, ranges, and network policies.
• Restrict access to the self-provisioners cluster role.
• Create groups and assign users to groups.
• Use role-based access control (RBAC) to grant permissions to groups.
Before You Begin
As the student user on the workstation machine, use the lab command to prepare your
system for this exercise.
[student@workstation ~]$ lab start compreview-review
The lab command copies the exercise files to the ~/DO280 directory and creates the
following users:
• do280-support
• do280-platform
• do280-presenter
• do280-attendee
The goal, as the cluster administrator, is to configure a dedicated cluster to host workshops
on different topics.
Each workshop requires a project, so that workshops are isolated from each other.
You must set up the cluster so that when the presenter creates a workshop project, the
project gets a base configuration.
The presenter must be mostly self-sufficient to administer a workshop with little help from
the workshop support team.
The workshop support team must deploy applications that administer workshops and that
enhance the workshop experience. You set up a project and the applications for this purpose
on a second lab.



2. Create the following groups and add a user as specified in the following table.
Group 				User
workshop-support 	do280-support
presenters 			do280-presenter
platform 			do280-platform

<b> Create the workshop-support group. </b>

[mahsan@r9 ~]$ oc adm groups new workshop-support
group.user.openshift.io/workshop-support created

Add the do280-support user to the workshop-support group

[mahsan@r9 ~]$ oc adm groups add-users workshop-support do280-support
group.user.openshift.io/workshop-support added: "do280-support"

<b> Create the presenters group. </b>

[mahsan@r9 ~]$ oc adm groups new presenters
group.user.openshift.io/presenters created

<b>Add the do280-presenter user to the presenters group.</b>

[mahsan@r9 ~]$ oc adm groups add-users presenters do280-presenter
group.user.openshift.io/presenters added: "do280-presenter"

<b> Create the platform group </b>
[mahsan@r9 ~]$ oc adm groups new platform
group.user.openshift.io/platform created
[mahsan@r9 ~]$

<b>Add the do280-platform user to the platform group </b>
[mahsan@r9 ~]$ oc adm groups add-users platform do280-platform
group.user.openshift.io/platform added: "do280-platform"
[mahsan@r9 ~]$ oc get groups
NAME               USERS
platform           do280-platform
presenters         do280-presenter
workshop-support   do280-support


[mahsan@r9 comprehensive-review]$ vim role.yaml
[mahsan@r9 comprehensive-review]$ oc create -f role.yaml
role.rbac.authorization.k8s.io/pod-reader created
[mahsan@r9 comprehensive-review]$ cat role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
[mahsan@r9 comprehensive-review]$


<b> Grant the admin cluster role to the workshop-support group </b>
[mahsan@r9 comprehensive-review]$ oc adm policy add-cluster-role-to-group admin workshop-support
clusterrole.rbac.authorization.k8s.io/admin added: "workshop-support"


<b> Grant the manage-groups cluster role to the workshop-support group </b>
[mahsan@r9 comprehensive-review]$ oc adm policy add-cluster-role-to-group manage-groups workshop-support
Warning: role 'manage-groups' not found
clusterrole.rbac.authorization.k8s.io/manage-groups added: "workshop-support"
[mahsan@r9 comprehensive-review]$ oc adm policy add-cluster-role-to-group manage-groups workshop-support
clusterrole.rbac.authorization.k8s.io/manage-groups added: "workshop-support"
[mahsan@r9 comprehensive-review]$

<b> Create a cluster role binding to assign the cluster-admin cluster role to the platform group.</b>
[mahsan@r9 comprehensive-review]$ oc adm policy add-cluster-role-to-group cluster-admin platform
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "platform"

<b>
Allow only the platform, workshop-support and presenters groups to create
projects, by editing the self-provisioner cluster role. Enforce that only users from
these groups can create projects. Also, make this change permanent by setting the
rbac.authorization.kubernetes.io/autoupdate annotation with the false value.</b>

<b>5.1. Use the oc edit command to edit the self-provisioners cluster role binding. </b>


[mahsan@r9 comprehensive-review]$ oc edit clusterrolebinding self-provisioners

[mahsan@r9 comprehensive-review]$ oc get clusterrolebinding self-provisioners
NAME                ROLE                           AGE
self-provisioners   ClusterRole/self-provisioner   106d
[mahsan@r9 comprehensive-review]$ oc get clusterrolebinding self-provisioners -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2024-12-03T08:13:44Z"
  name: self-provisioners
  resourceVersion: "5322"
  uid: 1a36a74a-94b9-450c-9121-c5c13eb18b46
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: self-provisioner
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:authenticated:oauth
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc edit clusterrolebinding self-provisioners

<b>
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "false"
  creationTimestamp: "2024-12-03T08:13:44Z"
  name: self-provisioners
  resourceVersion: "5322"
  uid: 1a36a74a-94b9-450c-9121-c5c13eb18b46
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: self-provisioner
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: platform
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: workshop-support
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: presenters
  </b>
  
[mahsan@r9 comprehensive-review]$ oc get users
NAME        UID                                    FULL NAME   IDENTITIES
developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
[mahsan@r9 comprehensive-review]$ oc login -u developer -p developer
Login successful.

You have one project on this server: "test-project"

Using project "test-project".
[mahsan@r9 comprehensive-review]$ oc new-project template-test
Error from server (Forbidden): You may not request a new project via this API.
[mahsan@r9 comprehensive-review]$



[mahsan@r9 comprehensive-review]$ oc login -u kubeadmin
Logged into "https://api.crc.testing:6443" as "kubeadmin" using existing credentials.

You have access to 66 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "test-project".
[mahsan@r9 comprehensive-review]$ oc new-project template-test
Now using project "template-test" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname



<b>
7. Create a template resource quota with the following specification.
quota value
limits.cpu 2
limits.memory 1Gi
requests.cpu 1500m
requests.memory 750Mi

</b>
  
  
  [mahsan@r9 comprehensive-review]$ cat quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
 name: workshop
 namespace: template-test
spec:
 hard:
   limits.cpu: 2
   limits.memory: 1Gi
   requests.cpu: 1500m
   requests.memory: 750Mi
[mahsan@r9 comprehensive-review]$ oc create -f quota.yaml
resourcequota/workshop created
[mahsan@r9 comprehensive-review]$ oc get quota
NAME       AGE   REQUEST                                           LIMIT
workshop   10s   requests.cpu: 0/1500m, requests.memory: 0/750Mi   limits.cpu: 0/2,limits.memory: 0/1Gi

<b> 
  
Create the workshop limit range with the following specification.
limit type 				value
max.cpu 				750m
max.mem 				750Mi
default.cpu 			500m
default.memory 			500Mi
defaulRequest.cpu 		100m
defaulRequest.memory 	250Mi  

</b>

[mahsan@r9 comprehensive-review]$ cat limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
 name: workshop
 namespace: template-test
spec:
 limits:
   - max:
       cpu: 750m
       memory: 750Mi
     default:
       cpu: 500m
       memory: 500Mi
     defaultRequest:
       cpu: 100m
       memory: 250Mi
     type: Container
[mahsan@r9 comprehensive-review]$ oc create -f limitrange.yaml
limitrange/workshop created
[mahsan@r9 comprehensive-review]$ oc get limitranges
NAME       CREATED AT
workshop   2025-03-19T17:25:42Z
[mahsan@r9 comprehensive-review]$ oc describe limitranges workshop
Name:       workshop
Namespace:  template-test
Type        Resource  Min  Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---    ---------------  -------------  -----------------------
Container   cpu       -    750m   100m             500m           -
Container   memory    -    750Mi  250Mi            500Mi          -
[mahsan@r9 comprehensive-review]$


<b>
Create a network policy to accept traffic from within the workshop project or from outside
the cluster. To identify the workshop project traffic, label the template-test namespace
with the workshop=template-test label.
9.1. Use the oc create deployment command to create a deployment without resource
specifications.
</b>
~

[mahsan@r9 comprehensive-review]$ oc create deployment test-workload --image  quay.io/redhattraining/hello-world-nginx:v1.0
deployment.apps/test-workload created
[mahsan@r9 comprehensive-review]$ oc get deployment
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
test-workload   0/1     1            0           5s

[mahsan@r9 comprehensive-review]$ oc get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
test-workload-7f649c44d5-hbldz   1/1     Running   0          43s   10.217.0.79   crc    <none>           <none>


[mahsan@r9 comprehensive-review]$ oc debug --to-namespace="default" -- curl -s http://10.217.0.79:8080
Starting pod/image-debug-72vpm ...
<html>
  <body>
    <h1>Hello, world from nginx!</h1>
  </body>
</html>

Removing debug pod ...
[mahsan@r9 comprehensive-review]$


[mahsan@r9 comprehensive-review]$ oc label ns template-test workshop=template-test
namespace/template-test labeled
[mahsan@r9 comprehensive-review]$ oc get ns template-test --show-labels
NAME            STATUS   AGE   LABELS
template-test   Active   50m   kubernetes.io/metadata.name=template-test,pod-security.kubernetes.io/audit-version=latest,pod-security.kubernetes.io/audit=restricted,pod-security.kubernetes.io/warn-version=latest,pod-security.kubernetes.io/warn=restricted,workshop=template-test
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ cat networkpolicy.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: workshop
  namespace: template-test
spec:
  podSelector: {}
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            workshop: template-test
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
			
			
[mahsan@r9 comprehensive-review]$ cat networkpolicy.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: workshop
  namespace: template-test
spec:
  podSelector: {}
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            workshop: template-test
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
[mahsan@r9 comprehensive-review]$ oc create -f networkpolicy.yaml
networkpolicy.networking.k8s.io/workshop created
[mahsan@r9 comprehensive-review]$ oc get netpol
NAME       POD-SELECTOR   AGE
workshop   <none>         4s
[mahsan@r9 comprehensive-review]$ oc describe netpol workshop
Name:         workshop
Namespace:    template-test
Created on:   2025-03-19 10:56:39 -0700 PDT
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      NamespaceSelector: workshop=template-test
    From:
      NamespaceSelector: network.openshift.io/policy-group=ingress
  Not affecting egress traffic
  Policy Types: Ingress
[mahsan@r9 comprehensive-review]$


[mahsan@r9 comprehensive-review]$ oc debug --to-namespace="default" -- curl -s http://10.217.0.79:8080
Starting pod/image-debug-gzdxj ...

Removing debug pod ...
error: non-zero exit code from debug container

[mahsan@r9 comprehensive-review]$ oc debug --to-namespace="default" -- curl -s http://10.217.0.79:8080
Starting pod/image-debug-gzdxj ...

Removing debug pod ...
error: non-zero exit code from debug container
[mahsan@r9 comprehensive-review]$ oc debug --to-namespace="template-test" -- curl -s http://10.217.0.79:8080
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "debug" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "debug" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "debug" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "debug" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
Starting pod/image-debug-b456s ...
<html>
  <body>
    <h1>Hello, world from nginx!</h1>
  </body>
</html>

Removing debug pod ...
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc adm create-bootstrap-project-template -o yaml > project-template.yaml
[mahsan@r9 comprehensive-review]$ cat project-template.yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    creationTimestamp: null
    name: ${PROJECT_NAME}
  spec: {}
  status: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: null
    name: admin
    namespace: ${PROJECT_NAME}
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc get resourcequota/workshop limitrange/workshop networkpolicy/workshop -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    creationTimestamp: "2025-03-19T17:16:01Z"
    name: workshop
    namespace: template-test
    resourceVersion: "87882"
    uid: d729dae9-b484-487f-941f-ba894c80db4c
  spec:
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
  status:
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
    used:
      limits.cpu: 500m
      limits.memory: 500Mi
      requests.cpu: 100m
      requests.memory: 250Mi
- apiVersion: v1
  kind: LimitRange
  metadata:
    creationTimestamp: "2025-03-19T17:25:42Z"
    name: workshop
    namespace: template-test
    resourceVersion: "82633"
    uid: 5af47f72-ed01-495a-9aef-b8243fa2292a
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 500Mi
      defaultRequest:
        cpu: 100m
        memory: 250Mi
      max:
        cpu: 750m
        memory: 750Mi
      type: Container
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    creationTimestamp: "2025-03-19T17:56:39Z"
    generation: 1
    name: workshop
    namespace: template-test
    resourceVersion: "87065"
    uid: 7feef7ce-0947-435c-9591-548b9c195af8
  spec:
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            workshop: template-test
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
    podSelector: {}
    policyTypes:
    - Ingress
kind: List
metadata:
  resourceVersion: ""
[mahsan@r9 comprehensive-review]$



[mahsan@r9 comprehensive-review]$ oc get resourcequota/workshop limitrange/workshop networkpolicy/workshop -o yaml >> project-template.yaml
[mahsan@r9 comprehensive-review]$ cat project-template.yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    creationTimestamp: null
    name: ${PROJECT_NAME}
  spec: {}
  status: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: null
    name: admin
    namespace: ${PROJECT_NAME}
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
apiVersion: v1
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    creationTimestamp: "2025-03-19T17:16:01Z"
    name: workshop
    namespace: template-test
    resourceVersion: "87882"
    uid: d729dae9-b484-487f-941f-ba894c80db4c
  spec:
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
  status:
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
    used:
      limits.cpu: 500m
      limits.memory: 500Mi
      requests.cpu: 100m
      requests.memory: 250Mi
- apiVersion: v1
  kind: LimitRange
  metadata:
    creationTimestamp: "2025-03-19T17:25:42Z"
    name: workshop
    namespace: template-test
    resourceVersion: "82633"
    uid: 5af47f72-ed01-495a-9aef-b8243fa2292a
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 500Mi
      defaultRequest:
        cpu: 100m
        memory: 250Mi
      max:
        cpu: 750m
        memory: 750Mi
      type: Container
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    creationTimestamp: "2025-03-19T17:56:39Z"
    generation: 1
    name: workshop
    namespace: template-test
    resourceVersion: "87065"
    uid: 7feef7ce-0947-435c-9591-548b9c195af8
  spec:
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            workshop: template-test
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
    podSelector: {}
    policyTypes:
    - Ingress
kind: List
metadata:
  resourceVersion: ""
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ sed -i 's/template-test/${PROJECT_NAME}/g' project-template.yaml

[mahsan@r9 comprehensive-review]$ oc create -f project-template.yaml -n openshift-config
template.template.openshift.io/project-request created
[mahsan@r9 comprehensive-review]$ cat project-template.yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    name: ${PROJECT_NAME}
    labels:
      workshop: ${PROJECT_NAME}
  spec: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: admin
    namespace: workshop
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: workshop
    namespace: ${PROJECT_NAME}
  spec:
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
    hard:
      limits.cpu: "2"
      limits.memory: 1Gi
      requests.cpu: 1500m
      requests.memory: 750Mi
    used:
      limits.cpu: 500m
      limits.memory: 500Mi
      requests.cpu: 100m
      requests.memory: 250Mi
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: workshop
    namespace: ${PROJECT_NAME}
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 500Mi
      defaultRequest:
        cpu: 100m
        memory: 250Mi
      max:
        cpu: 750m
        memory: 750Mi
      type: Container
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: workshop
    namespace: ${PROJECT_NAME}
  spec:
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            workshop: ${PROJECT_NAME}
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
    podSelector: {}
    policyTypes:
    - Ingress
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc describe template project-request -n openshift-config
Name:           project-request
Namespace:      openshift-config
Created:        About a minute ago
Labels:         <none>
Annotations:    <none>

Parameters:
    Name:       PROJECT_NAME
    Required:   false
    Value:      <none>

    Name:       PROJECT_DISPLAYNAME
    Required:   false
    Value:      <none>

    Name:       PROJECT_DESCRIPTION
    Required:   false
    Value:      <none>

    Name:       PROJECT_ADMIN_USER
    Required:   false
    Value:      <none>

    Name:       PROJECT_REQUESTING_USER
    Required:   false
    Value:      <none>


Object Labels:  <none>

Message:        <none>

Objects:
    Project.project.openshift.io                ${PROJECT_NAME}
    RoleBinding.rbac.authorization.k8s.io       admin
    ResourceQuota                               workshop
    LimitRange                                  workshop
    NetworkPolicy.networking.k8s.io             workshop
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc get projects.config.openshift.io cluster
NAME      AGE
cluster   106d
[mahsan@r9 comprehensive-review]$ oc get projects.config.openshift.io cluster -o yaml
apiVersion: config.openshift.io/v1
kind: Project
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    release.openshift.io/create-only: "true"
  creationTimestamp: "2024-12-03T08:02:21Z"
  generation: 1
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: 1de19896-228b-4a9d-913f-0d775bc4bebc
  resourceVersion: "1588"
  uid: 7e11232f-22f6-4f69-be46-5cae73c50b98
spec: {}
[mahsan@r9 comprehensive-review]$

apiVersion: config.openshift.io/v1
kind: Project
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    release.openshift.io/create-only: "true"
  creationTimestamp: "2024-12-03T08:02:21Z"
  generation: 1
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: 1de19896-228b-4a9d-913f-0d775bc4bebc
  resourceVersion: "1588"
  uid: 7e11232f-22f6-4f69-be46-5cae73c50b98
spec:
  projectRequestTemplate:
    name: project-request


oc edit projects.config.openshift.io cluster
Edit cancelled, no changes made.
[mahsan@r9 comprehensive-review]$ oc edit projects.config.openshift.io cluster
project.config.openshift.io/cluster edited
[mahsan@r9 comprehensive-review]$


 watch oc get pod -n openshift-apiserver

[mahsan@r9 comprehensive-review]$  oc get pod -n openshift-apiserver
NAME                         READY   STATUS    RESTARTS   AGE
apiserver-58d7c9b4dd-8qkjp   2/2     Running   0          2m49s
[mahsan@r9 comprehensive-review]$


[mahsan@r9 comprehensive-review]$ oc get resourcequotas  --all-namespaces
NAMESPACE                NAME                            AGE     REQUEST                                                              LIMIT
openshift-host-network   host-network-namespace-quotas   106d    count/daemonsets.apps: 0/0, count/deployments.apps: 0/0, pods: 0/0   limits.cpu: 0/0, limits.memory: 0/0
template-test            workshop                        4h44m   requests.cpu: 100m/1500m, requests.memory: 250Mi/750Mi               limits.cpu: 500m/2, limits.memory: 500Mi/1Gi
[mahsan@r9 comprehensive-review]$ oc get limitrange  --all-namespaces
NAMESPACE       NAME       CREATED AT
template-test   workshop   2025-03-19T17:25:42Z
[mahsan@r9 comprehensive-review]$ oc get networkpolicy  --all-namespaces
NAMESPACE       NAME       POD-SELECTOR   AGE
template-test   workshop   <none>         4h4m
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$ oc new-project  test-project2
Now using project "test-project2" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 comprehensive-review]$ oc get project test-project2 -o yaml
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  annotations:
    openshift.io/description: ""
    openshift.io/display-name: ""
    openshift.io/requester: kubeadmin
    openshift.io/sa.scc.mcs: s0:c26,c10
    openshift.io/sa.scc.supplemental-groups: 1000670000/10000
    openshift.io/sa.scc.uid-range: 1000670000/10000
  creationTimestamp: "2025-03-19T22:02:49Z"
  labels:
    kubernetes.io/metadata.name: test-project2
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
    <b>workshop: test-project2 </b>
  name: test-project2
  resourceVersion: "122013"
  uid: 698d4588-c5b7-43c5-9167-4e52bd54ac06
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
[mahsan@r9 comprehensive-review]$

[mahsan@r9 comprehensive-review]$  oc adm policy add-role-to-group edit d0280-attendees -n test-project2
Warning: Group 'd0280-attendees' not found
clusterrole.rbac.authorization.k8s.io/edit added: "d0280-attendees"
[mahsan@r9 comprehensive-review]$  oc adm policy add-role-to-group edit d0280-attendees -n test-project2
clusterrole.rbac.authorization.k8s.io/edit added: "d0280-attendees"
[mahsan@r9 comprehensive-review]$


</pre>


