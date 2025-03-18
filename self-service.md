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



</pre>

