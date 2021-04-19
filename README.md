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

# Kubectl Artifacts

This package contains a handful of kubectl artifacts intended to ease the
burden of more securely operating a kubernetes cluster with little or no authentication
framework configured.

# psp.yaml

To install the PodSecurityPolicy, run the following command:

    kubectl apply -f psp.yaml

If you wish to change the name of the pod security policy, do so with this command:

    sed -e s/restricted/NEW-PSP-NAME/ psp.yaml | kubectl apply -f -

If you change the name of the PSP, you will need to manually edit the
`kubectl-createns` plugin to reference the correct PodSecurityPolicy name.

# Installation

Each of the kubectl plugins in this directory need to be installed into
a directory in your search path ($PATH) to be usable. An example
installation command is:

    mkdir -p ~/bin
    cp bin/kubectl-* ~/bin/

Alternatively, after making the directory, you can make symlinks to the
scripts to allow for easier editing and development like so:

    for b in $(pwd)/bin/kubectl-*; do ln -s $b ~/bin/; done

If ~/bin is not in your search path, it can be added with the following:

    export PATH=$PATH:~/bin

Lastly, if you wish to make this addition to your search path stick
between sessions, you can persist it with:

    echo "PATH=\$PATH:~/bin" >> ~/.profile

# Invocation

All of the plugins created here can be run in debug mode by simply
exporting the environment variable DEBUG to any value.

    export DEBUG=1

Alternatively, it can be turned on for a single run of the command by
specifying environment variables in the normal fashion:

    DEBUG=1 kubectl getdeployertoken

Further documentation can be found in the bin directory.

# Workflow

These kubectl plugins are intended to automate the following steps:

## Cluster preparation

The PodSecurityPolicy needs to be applied to the cluster with:

    kubectl apply -f psp.yaml

## kubectl-createns

Creates a namespace and associated accounts, roles, permissions, and bindings in said namespace as
follows:

1. Creating the kubernetes namespace
2. Creating a deployer service account in the namespace.
3. Creating a runner service account in the namespace.
4. Creating a deployer role in the namespace with sufficient privileges to
create deployments and ReplicaSets but not pods and utilizing the PodSecurityPolicy.
5. Creating a runner role in the namespace with pod creating capabilities and the ability to use the PodSecurityPolicy.
6. Creating associated RoleBindings for the deployer and its role as well as the runner and its role.

## kubectl-getdeployertoken

1. Obtains the deployer token from the deployer ServiceAccount for distribution to tooling or developers.

## kubectl-login

1. Creates a kubeconfig cluster object. (optional)
2. Creates a kubeconfig credentials object containing the deployer ServiceAccountToken.
3. Creates a kubeconfig context object linking the credential and cluster objects.
4. Specifies the namespace to the kubeconfig object where the credentials are valid.

## kubectl-rotatetokens

Provides a mechanism to rotate ServiceAccount tokens in a namespace

1. Deletes one or more ServiceAccountToken secret in a namespace.

# External Artifacts

You may find the the [kubectl-whoami](https://github.com/rajatjindal/kubectl-whoami) plugin useful
for implementing and debugging these objects.
