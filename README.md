# wildfly-glow-tekton-pipeline

* Create secret for SSH KEY:
oc create secret generic my-ssh-credentials --from-file=known_hosts=/home/jdenise/.ssh/known_hosts --from-file=id_ed25519=/home/jdenise/.ssh/id_ed25519 --from-file=ssh-privatekey=/home/jdenise/.ssh/id_ed25519 --type=kubernetes.io/ssh-auth

* oc create -f pvc-sources-glow.yaml

* oc create -f quay-secret.yaml

* Add tekton and gitops operators

* oc new-project eap-service

* oc apply -f of all task/pipeline
oc apply -f wildfly-check-image-exists.yaml
oc apply -f wildfly-cleanup.yaml
oc apply -f wildfly-glow-generator-task.yaml
oc apply -f wildfly-glow-pipeline.yaml
oc apply -f wildfly-identify-branch.yaml
oc apply -f wildfly-import-image.yaml
oc apply -f wildfly-setup.yaml

* Create role binding to allow tocreate image Stream in openshift-gitops
oc apply -f rolebinding.yaml

Can be done in web console:
User Management/RoleBindings
Create Role binding
Cluster wide role binding
Name: pipeline-eap-server
Role Name: admin
Subject ServiceAccount
Subject Namespace: eap-service
Subject Name: pipeline

* Start the first demo
oc create -f demo-build.yaml