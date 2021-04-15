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

# Roles and RoleBindings

These are the yaml templates that are used to create the role and
RoleBindings in kubectl-createns.

Should you wish to manually apply these roles/bindings/serviceaccounts,
there is a file called `all-in-one.yaml` that will apply the PSP and all
necessary artifacts in the current namespace (or whichever namespace is
specified on the command-line with the `-n` flag). Be sure to edit the
`restricted` PSP name if you've renamed the PSP. If you wish to rename
the deployer and/or runner roles, you can do so with this:

    sed -e s/runner/NEW-RUNNER-NAME/ -e s/deployer/NEW-DEPLOYER-NAME/ \
        all-in-one.yaml | kubectl apply -f -
