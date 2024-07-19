# wildfly-glow-tekton-pipeline

A pipeline that leverages [WildFly Glow](https://github.com/wildfly/wildfly-glow) [OpenShift dry-run mode](https://docs.wildfly.org/wildfly-glow/#_openshift_dry_run_mode) to build optimized images and deploy resources.

For more details on this pipeline, you can read this [document](pipeline-details.md).

In order to run it in your own context, you would have to upate the pipeline runs with your own git repositories, container image and credentials.

# OpenShift sandbox setup and pipeline runs

* Create secrets:

oc create secret generic my-ssh-credentials --from-file=known_hosts=/home/jdenise/.ssh/known_hosts --from-file=id_ed25519=/home/jdenise/.ssh/id_ed25519 --from-file=ssh-privatekey=/home/jdenise/.ssh/id_ed25519 --type=kubernetes.io/ssh-auth

oc create -f quay-secret.yaml

* Create the pipeline, tasks and volume

oc apply -f pipeline-resources.yaml

* Run the pipeline for the kitchensink application

oc create -f pipeline-runs/demo-build.yaml

* Run the pipeline for an HA application

oc create -f pipeline-runs/demo-ha-build.yaml

* Run the pipeline for the todo-backend + postgresql application

oc create -f pipeline-runs/demo-db-build.yaml

#  Argo CD based deployment, requires a managed OpenShift cluster setup and pipeline runs

oc new-project wildfly-glow-pipeline

* Create secrets and volumes:

oc create secret generic my-ssh-credentials --from-file=known_hosts=/home/jdenise/.ssh/known_hosts --from-file=id_ed25519=/home/jdenise/.ssh/id_ed25519 --from-file=ssh-privatekey=/home/jdenise/.ssh/id_ed25519 --type=kubernetes.io/ssh-auth

oc create -f quay-secret.yaml

* Add tekton and gitops operators

* Create the pipeline

oc apply -f pipeline-resources.yaml

* Create role binding to allow to create image Stream in openshift-gitops

oc apply -f rolebinding.yaml

* Create role binding to allow to create resources in the user-project

oc apply -f rolebinding-argocd.yaml

* Create the user project

oc new-project user-project

* Run the pipeline for the kitchensink application

oc create -f pipeline-runs/demo-build-argo-cd.yaml

* Run the pipeline for the todo-backend + postgresql application

oc create -f pipeline-runs/demo-db-build-argo-cd.yaml