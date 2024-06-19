# wildfly-glow-tekton-pipeline

* oc new-project eap-pipeline

* Create secret for SSH KEY:
oc create secret generic my-ssh-credentials --from-file=known_hosts=/home/jdenise/.ssh/known_hosts --from-file=id_ed25519=/home/jdenise/.ssh/id_ed25519 --from-file=ssh-privatekey=/home/jdenise/.ssh/id_ed25519 --type=kubernetes.io/ssh-auth

* oc create -f pvc-sources-glow.yaml

* oc create -f quay-secret.yaml

* Add tekton and gitops operators

* task/pipeline
oc apply -f wildfly-check-image-exists.yaml
oc apply -f wildfly-cleanup.yaml
oc apply -f wildfly-deploy.yaml
oc apply -f wildfly-glow-generator-task.yaml
oc apply -f wildfly-glow-pipeline.yaml
oc apply -f wildfly-provision.yaml
oc apply -f wildfly-setup.yaml

* Create role binding to allow to create image Stream in openshift-gitops
oc apply -f rolebinding.yaml

* Create role binding to allow to create resources in the user-project

oc apply -f rolebinding-argocd.yaml

* Create the user project

oc new-project user-project

* Start the first demo:
If argoCD:
oc create -f demo-build-argo-cd.yaml
If not:
oc create -f demo-build.yaml