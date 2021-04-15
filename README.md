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
a directory in your searchpath ($PATH) to be usable. An example
installation command is:

    mkdir -p ~/bin && cp bin/kubectl-* ~/bin/

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
specifying environment variables in the normal fasion:

    DEBUG=1 kubectl getdeployertoken

Furthur docmentation can be found in the bin directory.
