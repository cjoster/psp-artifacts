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

# kubectl-createns

This kubectl plugin is intended to automate the creation and population
of a new namespace with 'deployer' and 'runner' service accounts. It
configures the roles and pod-security-policies along with the
RoleBindings required to tie them all together.

Currently these Roles, RoleBindings, and PodSecurityPolicies are housed
in the script as hard-coded variables. A better mechanism to reference
more mutable and maintainable kubernetes objects is needed.

Invocation is simple:

    kubectl createns mynamespace

Help output is available also with `-h` or `help`:

    Usage:
        kubectl createns namespace [ --fixup ]
        
        namespace: Name of the namespace to create
        --fixup  : Attempt to deploy/modify ServiceAccounts, Roles,
        	 : and rolebindings even if namespace already exists.

# kubectl-getdeployertoken

Kubernetes plugin intended to simplify obtaining the deployer token
created by `kubectl createns`. Invoke as:

    kubectl getdeployertoken [ -n namespace ]

Example output is:

    kubectl getdeployertoken
    
       YOUR DEPLOYER TOKEN IS:
    
    eyJhbGciOiJSUzI1NiIsImtpZCI6InBPYWZoak9WaDhCaVdUN3NaU....

Help output is available also with `-h` or `help`:

    Usage:
        kubectl getdeployertoken [ -n namespace ]
        
        -n namespace: Namespace from which to display the deployer token.
        Displays deployer token from current namespace it not
        specified.

# kubectl-rotatetokens

This kubectl plugin assists with rotating service account tokens in a
namespace. The script does so by simply deleting all secrets in the
namespace that are of type 'kubernetes.io/service-account-token',
causing the ServiceAccount controller to generate new tokens.

The plugin can be invoked to specify a single token:

    kubectl rotatetokens default -n mynamespace

or it can be invoked on an entire namespace:

    kubectl rotatetokens -n mynamespace --all

Not specifying a namespace will cause all tokens

It does not have a -A option for obvious reasons. If you wish to
rotate all service account tokens cluster-wide, you can do so with
this command:

    for ns in $(kubectl get namespaces --no-headers | awk '{print $1}'); do \
        kubectl rotatetokens -n "${ns}" --all; done

It is strongly recommended that you not do this.

Help output is available also with `-h` or `help`:

    Usage:
        kubectl rotatetokens ( service-account-name | --all ) [ -n namespace ]
        
        service-account-name: Name of service account whose token is to be rotated.
        -n namespace        : Namespace in which to rotate service-account-names's token.
        --all               : Rotate all tokens in the specified namespace. Rotates all
                              tokens in the current namespace if -n is not specified.

# kubectl-login

Kubectl plugin intended to automate the population of a local kubeconfig
file with the deployer token for a given namespace.

Help output is available also with `-h` or `help`:

    Usage:
        kubectl login --name=clustername [ --ca-file=ca.crt | --extract-ca [ --no-verify-ca ] ]
        --namespace=namespace --token=token [ http://cluster-url:port ]
        
        --name        : Friendly (arbitrary) name of the cluster into to which you want to
                        login. Used to construct the cluster context name and credential name.
        --namespace   : The namespace in which this deployer service account exists.
        --ca-file     : A file containing the CA certificate for the cluster.
        --extract-ca  : Extracts the certificate authority from the cluster rather than
                        supplying it on the command line.
        --no-verify-ca: Does not present the CA certificate extracted with --extract-ca to
                        the user for verification.
        --token       : The service account token provided to you by your cluster administrator.
        
        http://cluster-url:port: URL for cluster API. Optional if clustername already exists.
