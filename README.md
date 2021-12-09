# Workshop JBoss EAP on Openshift

## Operation

1. Install OC Client.
    - Download OC: http://d3s3zqyaz8cp2d.cloudfront.net/pub/openshift-v4/clients/ocp/4.6.20/openshift-client-linux-4.6.20.tar.gz
      * [oc client folder](./tools/oc-client/)
    - Copy on your PATH
    - Run: oc version

2. Cluster login:
    - Go to cluster and login as cluster-admin: https://console-openshift-console.apps.cluster-58b1.dynamic.opentlc.com/dashboards
      * Cluster admin: user: admin , password: pU8uNdrpviUgbRrH
      * Users:  user1 .. user10, password: BAqpz5uVc33Quqf5
    - Click on user button (upper rigth), and click 'Copy login command'
    - Copy the login command shown.
    - Paste on your terminal: oc login --token=TOKEN --server=https://api.cluster-58b1.dynamic.opentlc.com:6443


3. Basic Operations:
    - Whoami: 
      * oc whoami
    - Projects / Namespaces:
      * oc projects (list all projects)
      * oc project PROJECT_NAME: oc new-project eap-demo
    - New application:
      * oc new-build IMAGE_NAME~SOURCE_CODE --name BUILD_NAME: oc new-build nodejs~https://github.com/cesarvr/hello-world-nodejs --name nodejs-build
      * oc new-app nodejs-build --name=my-nodejs-app
      * oc expose service/my-nodejs-app  (create a route)
    - Get / Describe Objects:
      * oc get all
      * oc get OBJECT_TYPE (list): oc get bc
      * oc get OBJECT_TYPE OBJECT_NAME -o yaml -n PROJECT_NAME: 
        * oc get bc nodejs-build -o yaml
        * oc get bc nodejs-build -o yaml > nodejs-build.yaml
    - Edit Objects:
      * oc edit OBJECT_TYPE OBJECT_NAME: oc edit bc nodejs-build
    - Create / Delete objects from file:
      * oc create -f FILE_NAME: oc create -f nodejs-build.yaml
      * oc delete -f FILE_NAME: oc delete -f nodejs-build.yaml


## Deploy

## Monitoring

## Troubleshooting




# -------------------

# JBoss Enterprise Application Platform continuous delivery OpenShift container image

NOTE: Extends link:https://github.com/jboss-container-images/jboss-eap-7-image[JBoss EAP 7.3 container image]

## Importing Imagestreams and Templates

You should have the 'oc' tools available and be logged in to your OpenShift instance. For more details on how to do this, please consult the OpenShift documentation.
For example, for OpenShift Online, see: https://docs.openshift.com/online/pro/cli_reference/get_started_cli.html
[source, bash]
----
for resource in eap73-image-stream.json \
  eap73-amq-persistent-s2i.json \
  eap73-amq-s2i.json \
  eap73-basic-s2i.json \
  eap73-https-s2i.json \
  eap73-sso-s2i.json \
  eap73-third-party-db-s2i.json \
  eap73-tx-recovery-s2i.json
do
  oc replace --force -f https://raw.githubusercontent.com/jboss-container-images/jboss-eap-7-openshift-image/7.3.x/templates/${resource}
done
----

If you have administrative access to the general `openshift` namespace and want the image streams and templates to be accessible by all projects, add -n openshift to the oc replace line of the command. For example:

[source, bash]
----
oc replace -n openshift --force -f ...
----

See the OpenShift documentation at https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/ for more details on importing templates.

#### Updating Existing Images
To update an to the most recent image:

[source, bash]
----
oc import-image jboss-eap73-openshift:7.3
----

Optionally include namespace to the import:
[source, bash]
----
oc -n myproject import-image jboss-eap73-openshift:7.3
----

#### Creating New Applications With Templates
Example:

[source, bash]
----
oc -n myproject new-app eap73-basic-s2i -p APPLICATION_NAME=eap73-basic-app
----

or, if a namespace was used with `-n` above:
[source, bash]
----
oc -n myproject new-app eap73-basic-s2i -p APPLICATION_NAME=eap73-basic-app -p IMAGE_STREAM_NAMESPACE=myproject
----

For more information on application creation and deployment see the OpenShift documentation at https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/ for more details on importing templates.

## Creating Secrets

Some of the included templates require the creation of secrets.

Example:
[source, bash]
----
$ keytool -genkey -keyalg RSA -alias eapdemo-selfsigned -keystore keystore.jks -validity 360 -keysize 2048
$ oc secrets new eap7-app-secret keystore.jks
----

Example secrets may also be found here: https://github.com/jboss-openshift/application-templates/tree/master/secrets

[source, bash]
----
oc create -n myproject -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/eap-app-secret.json
oc create -n myproject -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/eap7-app-secret.json
oc create -n myproject -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/sso-app-secret.json
----

Please refer to the EAP Documentation for additional details.

https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/ 

## License

See link:LICENSE[LICENSE] file.

