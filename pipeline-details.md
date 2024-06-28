
# WildFly Pipeline

A pipeline that leverages [WildFly Glow](https://github.com/wildfly/wildfly-glow) [OpenShift dry-run mode](https://docs.wildfly.org/wildfly-glow/#_openshift_dry_run_mode) to build optimized images and deploy resources.

## Pipeline built optimized images vs S2I built images

This pipeline approach allows for re-use of server images when building application images.
The pipeline build dissociates the application image in 2 pieces:

* The server image that, from the EAP runtime image (mainly ubi8-minimal + JDK), contains the provisioned server. A server image tag is identified by a hash of the generated provisioning configuration and channel.
* The application image that, from a server image adds the deployment + init/cli scripts.
In addition to this optimization that allows to save hundreds of megabytes, the pipeline will not rebuild application images if nothing changed in the application. Such “nothing to do” kind of build would imply the creation of a new application image with the S2I approach.

## Application Github repository

* github.com/jfdenise/my-war-2 ⇒ application branch containing war, …
* github.com/jfdenise/my-war-2#main_deployment ⇒ application branch containing k8s resources, this branch is created from the pipeline and monitored by Argo CD.

## Pipeline data GitHub repository

https://github.com/jfdenise/go-to-openshift-2 ⇒ These branches are were the pipeline stores the data it requires for optimized builds.

## Implementation

* This pipeline is based on WildFly 32.0.1.Final.
* WildFly Glow 1.1.0.Final, in particular the new dry-run mode.
* Quay.io is used to store the images.
* Github is used to store the application, the k8s resources and the generated metadata.
* Tekton pipeline
* ArgoCD operator (OpenShift GitOps) is used to synchronize the deployment
* Galleon 6.0.0.Final CLI

### Pipeline details

A lot of optimizations are put in the pipeline to only rebuild application image and update deployment when needed.

#### WildFly Glow dry-run mode

* WildFly Glow CLI zip is retrieved from the internet and installed locally during pipeline execution.
* WildFly Glow scan produces:
    - Galleon provisioning.xml
    - Docker files
    - Packaged CLI script and init bash script (if any) ready to be installed in the application image
    - K8s resources

NOTE: This scan can produce errors (missing add-on that the user would have to fix in his repository and restart the pipeline).

#### Provisioning

Galleon CLI is used to provision the server.

#### Images creation

* Server and application images are built using Buildah, no dependency on OpenShift. 
* Server image is a simple copy of the provisioned server into the WildFly runtime image.
* Application image is FROM a server image and copy the deployments + packaged scripts.
* The image is referenced from the deployment yaml file by a tag that contains the image version, no ”latest”. 
This means that the deployment is updated by the pipeline each time a new app image is created.

#### Deployment branch conflicts handling

The pipeline commits and pushes changes to the xxx_deployment branch if the changes are applicable.
The user is allowed (and will) do changes to the deployments (add env variables,  secrets,…). So conflict handling must be handled.

Details:

* The pipeline references an internal git branch that contains a sequence of applicable commits (changes caused by generated yaml resources 
(eg: new produced application image referenced from deployment/statefulset).
* When a change is detected:
    - A commit is added to the internal git branch: COMMIT_1
    - A patch is generated from this commit.
    - The user deployment branch is pulled
    - The patch is attempted to be applied (--check) to the user deployment branch.
    - If the patch would apply successfully:
        - The COMMIT_1 is cherry-picked in the user deployment branch
        - The user deployment branch is pushed to the user repo
        - The internal branch is pushed to the internal repo
    - If the patch would apply with conflicts (for example the user as updated the image version in his deployment (for some reasons):
        - The COMMIT_1 is removed from the internal branch. 
        - The patch file is committed inside the user branch
        - An error commit message is used to advertise the conflict.
        - The branch is pushed to the user repo
        - The internal branch is not updated.
        - The pipeline aborts with an error. NOTE: The internal state of the application is not updated. This pipeline run is not recorded. Nevertheless, any images built during this failing run are valid and could be re-used in future pipeline runs.
The pipeline will abort until the changes are applicable to the user branch.

#### Measurement

The pipeline shows the following numbers:

* Full build with server and app images : 3 minutes
* App image only: 1:30 minutes
* Deployment impact only: 1 minute
* Do nothing: 30secs

