# wildfly-glow-tekton-pipeline

Create secret for SSH KEY:
oc create secret generic my-ssh-credentials --from-file=known_hosts=/home/jdenise/.ssh/known_hosts --from-file=id_ed25519=/home/jdenise/.ssh/id_ed25519 --from-file=ssh-privatekey=/home/jdenise/.ssh/id_ed25519 --type=kubernetes.io/ssh-auth
