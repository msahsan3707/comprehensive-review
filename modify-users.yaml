<pre>
<h1> Modify Users </h1>

 [mahsan@r9 ~]$ oc get identities.user.openshift.io
NAME                  IDP NAME    IDP USER NAME   USER NAME   USER UID
developer:developer   developer   developer       developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin   developer   kubeadmin       kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc get secrets -n openshift-config
NAME                                      TYPE                             DATA   AGE
builder-dockercfg-7r7fz                   kubernetes.io/dockercfg          1      109d
default-dockercfg-676lc                   kubernetes.io/dockercfg          1      109d
deployer-dockercfg-566hs                  kubernetes.io/dockercfg          1      109d
etcd-client                               kubernetes.io/tls                2      110d
htpass-secret                             Opaque                           1      109d
initial-service-account-private-key       Opaque                           1      110d
login-template                            Opaque                           1      109d
pull-secret                               kubernetes.io/dockerconfigjson   1      110d
webhook-authentication-integrated-oauth   Opaque                           1      110d
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc get secrets -n openshift-config
NAME                                      TYPE                             DATA   AGE
builder-dockercfg-7r7fz                   kubernetes.io/dockercfg          1      109d
default-dockercfg-676lc                   kubernetes.io/dockercfg          1      109d
deployer-dockercfg-566hs                  kubernetes.io/dockercfg          1      109d
etcd-client                               kubernetes.io/tls                2      110d
htpass-secret                             Opaque                           1      109d
initial-service-account-private-key       Opaque                           1      110d
login-template                            Opaque                           1      109d
pull-secret                               kubernetes.io/dockerconfigjson   1      110d
webhook-authentication-integrated-oauth   Opaque                           1      110d
[mahsan@r9 ~]$ oc extract secret/htpass-secret -n openshift-config --to /tmp
/tmp/htpasswd
[mahsan@r9 ~]$ ls -ltr /tmp/
total 4

rw------- 1 mahsan mahsan 141 Mar 23 12:26 htpasswd

<b> yum install httpd-tools </b>

[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd bob redhat
Adding password for user bob
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd qwerty redhat
Adding password for user qwerty
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd qwerty redhat
Updating password for user qwerty
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd john redhat
Adding password for user john
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd armstrong redhat
Adding password for user armstrong
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd natasha  redhat
Adding password for user natasha
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd harry  redhat
Adding password for user harry
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd susan  redhat
Adding password for user susan
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd mahsan  redhat
Adding password for user mahsan
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd new_admin  redhat
Adding password for user new_admin
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd new_developer  redhat
Adding password for user new_developer

[mahsan@r9 ~]$ oc set data secret/htpass-secret --from-file htpasswd=/tmp/htpasswd  -n openshift-config
secret/htpass-secret data updated
[mahsan@r9 ~]$ watch oc get all -n openshift-authentication
[mahsan@r9 ~]$ watch oc get pods -n openshift-authentication

[mahsan@r9 ~]$ oc login -u armstrong -p redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc login -u new_admin
Logged into "https://api.crc.testing:6443" as "new_admin" using existing credentials.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc get node
Error from server (Forbidden): nodes is forbidden: User "new_admin" cannot list resource "nodes" in API group "" at the cluster scope
[mahsan@r9 ~]$ oc login -u kubeadmin
Logged into "https://api.crc.testing:6443" as "kubeadmin" using existing credentials.

You have access to 66 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
[mahsan@r9 ~]$ oc get node
NAME   STATUS   ROLES                         AGE    VERSION
crc    Ready    control-plane,master,worker   110d   v1.30.6
[mahsan@r9 ~]$

<b> change the passwd </b>

[mahsan@r9 ~]$ oc extract secret/htpass-secret -n openshift-config --to /tmp --confirm
/tmp/htpasswd

[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd new_admin tripod001
Updating password for user new_admin
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd mahsan tripod001
Updating password for user mahsan
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc set data secret/htpass-secret --from-file htpasswd=/tmp/htpasswd  -n openshift-config
secret/htpass-secret data updated
[mahsan@r9 ~]$  oc get all -n openshift-authentication
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                   READY   STATUS    RESTARTS   AGE
pod/oauth-openshift-7d66cfc946-hhmg8   1/1     Running   0          9m17s

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/oauth-openshift   ClusterIP   10.217.4.49   <none>        443/TCP   110d

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oauth-openshift   1/1     1            1           110d

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/oauth-openshift-6fcc9dd448   0         0         0       110d
replicaset.apps/oauth-openshift-78df6fd57d   0         0         0       110d
replicaset.apps/oauth-openshift-7d66cfc946   1         1         1       9m17s
replicaset.apps/oauth-openshift-7fcd5d96ff   0         0         0       110d
replicaset.apps/oauth-openshift-b468b8d84    0         0         0       88d
replicaset.apps/oauth-openshift-d494d9776    0         0         0       110d
replicaset.apps/oauth-openshift-d66d84594    0         0         0       109d

NAME                                       HOST/PORT                          PATH   SERVICES          PORT   TERMINATION            WILDCARD
route.route.openshift.io/oauth-openshift   oauth-openshift.apps-crc.testing          oauth-openshift   6443   passthrough/Redirect   None
[mahsan@r9 ~]$

[mahsan@r9 ~]$ watch  oc get pods -n openshift-authentication
[mahsan@r9 ~]$ oc login -u new_admin -p tripod001
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get users
NAME        UID                                    FULL NAME   IDENTITIES
armstrong   2860e2fc-9b56-4ce0-9aa9-47c02a72ed7e               developer:armstrong
developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
mahsan      f8c0afbb-429f-419a-b9c9-c78f7b363467               developer:mahsan
new_admin   cb8aeb3c-e77e-4bd5-8ce2-0425f155aad3               developer:new_admin
[mahsan@r9 ~]$ oc get identity
NAME                  IDP NAME    IDP USER NAME   USER NAME   USER UID
developer:armstrong   developer   armstrong       armstrong   2860e2fc-9b56-4ce0-9aa9-47c02a72ed7e
developer:developer   developer   developer       developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin   developer   kubeadmin       kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd
developer:mahsan      developer   mahsan          mahsan      f8c0afbb-429f-419a-b9c9-c78f7b363467
developer:new_admin   developer   new_admin       new_admin   cb8aeb3c-e77e-4bd5-8ce2-0425f155aad3
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc extract secret/htpass-secret -n openshift-config --to /tmp --confirm
/tmp/htpasswd
[mahsan@r9 ~]$ htpasswd  -D  /tmp/htpasswd mahsan
Deleting password for user mahsan
[mahsan@r9 ~]$ htpasswd  -D  /tmp/htpasswd new_admin
Deleting password for user new_admin
[mahsan@r9 ~]$ cat /tmp/htpasswd
developer:$2a$10$EvIsXb9Ayct2RAr9PTrr4.b0ewHvv78pexbJayKHZD1eyQnWTqZeO
kubeadmin:$2a$10$ZX76A.vm0dkUo0Dyj5BmVOtfhbQvt9FAmSn3pfKrhlsGUoaB1n6hWbob:$2y$05$LBJS7wTQ3p0O3eswNj1WYelE/uVGIYlcvyDdx/Hv2yWgFju6J5VxC
qwerty:$2y$05$ceCzNO87uR94ilpXIemeouE.5sUZ51uNMSrha6GiNzkT7xn25xdIS
john:$2y$05$Zt0Lo48tG8ZWr/ZQ4t/yDuCM9H2wP8J63Bp/oPeMr9HznknsG0R2m
armstrong:$2y$05$1lclz33eF..APrc9IVNLieo1304f4TidgNI/NtJP8C.Q.1h7uAUjO
natasha:$2y$05$gBW5k7GsQgIxVY/NNYaIzujPyE.fud4V2kusiiRFf2g82WDHAfEtS
harry:$2y$05$RVnrrgdtEh3sZLDsdT7kWODr9T68.8WcjKL.oZHna.2b41DngUhHG
susan:$2y$05$wZsC.bg0HcGY42hvbYJc5uWSe5RZA2HVn9jXhDy3IP8CVa/bau.ka
new_developer:$2y$05$dbcMT4zyBDoy.63pSs04L.22.znrkPfPVA2mwb6k./fQNpPozHTY6
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc set data secret/htpass-secret --from-file htpasswd=/tmp/htpasswd  -n openshift-config
secret/htpass-secret data updated
[mahsan@r9 ~]$ oc get pods -n openshift-authentication
NAME                             READY   STATUS    RESTARTS   AGE
oauth-openshift-9b6bd6df-kzqnm   1/1     Running   0          4m7s
[mahsan@r9 ~]$ watch oc get pods -n openshift-authentication
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get pods -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-56b9d579bf-2v6m7   1/1     Running   0          45s
[mahsan@r9 ~]$









</pre>

