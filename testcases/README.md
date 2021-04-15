<!--
###########################################################################
# Various Kubectl Artifacts (kubectl-login, kubectl-createns,             #
#    kubect-rotatetokens, kubectl-getdeployertoken                        #
# Copyright (C) 2021 CJ Oster (ocj@vmware.com)                            #
#                                                                         #
# This program is free software: you can redistribute it and/or modify    #
# it under the terms of the GNU Lesser General Public License as          #
# published by the Free Software Foundation, either version 3 of the      #
# License, or (at your option) any later version.                         #
#                                                                         #
# This program is distributed in the hope that it will be useful, but     #
# WITHOUT ANY WARRANTY; without even the implied warranty of              #
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser #
# General Public License for more details.                                #
#                                                                         #
# You should have received a copy of the GNU Lesser General Public        #
# License along with this program. If not, see                            #
# <https://www.gnu.org/licenses/>.                                        #
###########################################################################
-->

# Test Cases

In the two sub-folders contained in this folder are test cases that will
test out the the PodSecurityPolicy.

Use the `kubectl --as=...` to test these cases. You can also `kubectl login`
as the deployer once the tooling has been setup to try them out as well.
To use the service accounts from the command line, run kubectl as:

    kubectl --as=system:serviceaccount:production:runner ...
    kubectl --as=system:serviceaccount:production:deployer ...
    
    # In the case of testing out the pod-creator service account
    kubectl --as=system:serviceaccount:production:pod-creator ...

# Pods

Neither the `runner` nor the `deployer` ServiceAccount should be able to
apply any of the pods in the `pods` directory--but for different reasons.
The `runner` lacks privileges to do anything with the API and should fail
even on getting the existing pods.

Use the `kubectl config set-context` command to set your namespace to avoid
having to namespace all of your commands:

    kubectl config set-context --current --namespace=production

Test the `runner` and `deployer` service accounts as so:

    kubectl --as=system:serviceaccount:production:runner apply -f pods
    
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "/v1, Resource=pods", GroupVersionKind: "/v1, Kind=Pod"
    Name: "pod-valid", Namespace: "production"
    from server for: "pods/pod-valid.yaml": pods "pod-valid" is forbidden: User "system:serviceaccount:production:runner" cannot get resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "/v1, Resource=pods", GroupVersionKind: "/v1, Kind=Pod"
    Name: "pod-violation-escalation", Namespace: "production"
    from server for: "pods/pod-violation-escalation.yaml": pods "pod-violation-escalation" is forbidden: User "system:serviceaccount:production:runner" cannot get resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "/v1, Resource=pods", GroupVersionKind: "/v1, Kind=Pod"
    Name: "pod-violation-hostpid", Namespace: "production"
    from server for: "pods/pod-violation-hostpid.yaml": pods "pod-violation-hostpid" is forbidden: User "system:serviceaccount:production:runner" cannot get resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "/v1, Resource=pods", GroupVersionKind: "/v1, Kind=Pod"
    Name: "pod-violation-privileged", Namespace: "production"
    from server for: "pods/pod-violation-privileged.yaml": pods "pod-violation-privileged" is forbidden: User "system:serviceaccount:production:runner" cannot get resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "/v1, Resource=pods", GroupVersionKind: "/v1, Kind=Pod"
    Name: "pod-violation-root", Namespace: "production"
    from server for: "pods/pod-violation-root.yaml": pods "pod-violation-root" is forbidden: User "system:serviceaccount:production:runner" cannot get resource "pods" in API group "" in the namespace "production"
    
The `deployer`, however, will fail because the deployer can do most things
to pods except create and edit them.

    kubectl --as=system:serviceaccount:production:deployer apply -f pods
    
    Error from server (Forbidden): error when creating "pods/pod-valid.yaml": pods is forbidden: User "system:serviceaccount:production:deployer" cannot create resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when creating "pods/pod-violation-escalation.yaml": pods is forbidden: User "system:serviceaccount:production:deployer" cannot create resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when creating "pods/pod-violation-hostpid.yaml": pods is forbidden: User "system:serviceaccount:production:deployer" cannot create resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when creating "pods/pod-violation-privileged.yaml": pods is forbidden: User "system:serviceaccount:production:deployer" cannot create resource "pods" in API group "" in the namespace "production"
    Error from server (Forbidden): error when creating "pods/pod-violation-root.yaml": pods is forbidden: User "system:serviceaccount:production:deployer" cannot create resource "pods" in API group "" in the namespace "production"

An admin should be able to freely deploy any of the pods as they wish.

    kubectl apply -f pods
    
    pod/pod-valid created
    pod/pod-violation-escalation created
    pod/pod-violation-hostpid created
    pod/pod-violation-privileged created
    pod/pod-violation-root created
   
Verify the output:
 
    kubectl get pods
    
    NAME                       READY   STATUS    RESTARTS   AGE
    pod-valid                  1/1     Running   0          7s
    pod-violation-escalation   1/1     Running   0          7s
    pod-violation-hostpid      1/1     Running   0          6s
    pod-violation-privileged   1/1     Running   0          6s
    pod-violation-root         1/1     Running   0          5s

### Cleanup

    $ kubectl delete pods --all

## Pod-creator

There is a pod-creator.yaml file that will define a service account called
pod-creator. This ServiceAccount can be used to validate the PSP by itself. The pod-creator
is loaded like so:

    kubectl applyy -f pod-creator.yaml

Then you can attempt to create the pods:

    kubectl --as=system:serviceaccount:production:pod-creator apply -f pods
    
    pod/pod-valid created
    Error from server (Forbidden): error when creating "pods/pod-violation-escalation.yaml": pods "pod-violation-escalation" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.allowPrivilegeEscalation: Invalid value: true: Allowing privilege escalation for containers is not allowed]
    Error from server (Forbidden): error when creating "pods/pod-violation-hostpid.yaml": pods "pod-violation-hostpid" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.securityContext.hostPID: Invalid value: true: Host PID is not allowed to be used]
    Error from server (Forbidden): error when creating "pods/pod-violation-privileged.yaml": pods "pod-violation-privileged" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
    Error from server (Forbidden): error when creating "pods/pod-violation-root.yaml": pods "pod-violation-root" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.runAsUser: Invalid value: 0: running with the root UID is forbidden]

### Cleanup

    kubectl delete pods --all
    kubectl delete rolebinding pod-creator-rolebinding
    kubectl delete role pod-creator-role
    kubectl 

# Deployments

The `deployer` service account should not be able to deploy any of the
pods. It *should* be able to deploy all the deployments in the `deployments`
directory, but only the non-violating one should have pods come up.

    kubectl --as=system:serviceaccount:production:deployer apply -f deployments
    
    deployment.apps/deployment-valid created
    deployment.apps/deployment-violation-escalation created
    deployment.apps/deployment-violation-hostpid created
    deployment.apps/deployment-violation-privileged created
    deployment.apps/deployment-violation-root created

Verify the deployments loaded, but the violations don't come up:

    kubectl get deployments

    NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
    deployment-valid                  3/3     3            3           8s
    deployment-violation-escalation   0/3     0            0           8s
    deployment-violation-hostpid      0/3     0            0           8s
    deployment-violation-privileged   0/3     0            0           7s
    deployment-violation-root         0/3     0            0           7s

View the pods:

    kubectl get pods
    
    NAME                               READY   STATUS    RESTARTS   AGE
    deployment-valid-8fcfd5dc5-gqb9l   1/1     Running   0          16s
    deployment-valid-8fcfd5dc5-w7kff   1/1     Running   0          16s
    deployment-valid-8fcfd5dc5-xzggq   1/1     Running   0          16s

Then check the namespace events to see the PodSecurityPolicy failures:

    kubectl get ev  
    
    7s          Warning   FailedCreate        replicaset/deployment-violation-escalation-57bf49f5f    Error creating: pods "deployment-violation-escalation-57bf49f5f-" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.allowPrivilegeEscalation: Invalid value: true: Allowing privilege escalation for containers is not allowed]
    18s         Normal    ScalingReplicaSet   deployment/deployment-violation-escalation              Scaled up replica set deployment-violation-escalation-57bf49f5f to 3
    7s          Warning   FailedCreate        replicaset/deployment-violation-hostpid-557b7698b5      Error creating: pods "deployment-violation-hostpid-557b7698b5-" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.securityContext.hostPID: Invalid value: true: Host PID is not allowed to be used]
    17s         Normal    ScalingReplicaSet   deployment/deployment-violation-hostpid                 Scaled up replica set deployment-violation-hostpid-557b7698b5 to 3
    7s          Warning   FailedCreate        replicaset/deployment-violation-privileged-6b8454645c   Error creating: pods "deployment-violation-privileged-6b8454645c-" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
    17s         Normal    ScalingReplicaSet   deployment/deployment-violation-privileged              Scaled up replica set deployment-violation-privileged-6b8454645c to 3
    6s          Warning   FailedCreate        replicaset/deployment-violation-root-78fd9c7b87         Error creating: pods "deployment-violation-root-78fd9c7b87-" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.runAsUser: Invalid value: 0: running with the root UID is forbidden]


The `default` service account should not be able to do any of these
things.


