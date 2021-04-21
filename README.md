1) Install jenkins server in openshift using template
    oc new-project jenkins
    oc get templates -n openshift | grep jenkins
    oc new-app --as-deployment-config jenkins-ephemeral
    oc get pods
    oc get route
2) Create projects  for the pipeline 
    oc new-project movies-dev
    oc new-project movies-stage
3) configure the Jenkins Sync Plugin to monitor your project for pipelines

 From the Jenkins home page, click Manage Jenkins in the left menu of the Jenkins home page, and then click Configure System in the Manage Jenkins page to bring up the Jenkins global configuration page.
 Scroll down to the OpenShift Jenkins Sync section, and add the namespace
<<project-name>> to the Namespace field, next to the already existing
youruser-jenkins namespace. Separate the entries by a space, and do not use a
comma.

4) The service account associated with the Jenkins deployment must have the edit role for each project monitored by Jenkins
oc policy add-role-to-user  edit system:serviceaccount:jenkins:jenkins   -n <<jenkinsproject>>

oc policy add-role-to-user  edit system:serviceaccount:jenkins:jenkins   -n movies-dev
oc policy add-role-to-user  edit system:serviceaccount:jenkins:jenkins   -n movies-stage


5)  Create build config  in the dev project 
 oc project movies-dev
 oc create -f movies-bc.yaml

kind: "BuildConfig"
apiVersion: "build.openshift.io/v1"
metadata:
  name: "movies-pipeline" 
spec:
  source:
    contextDir: movies 
    git:
      uri: "https://github.com/abdelhamidfg/" 
      ref: "master"
  strategy:
    jenkinsPipelineStrategy:
     type: JenkinsPipeline 


6) Start the build 
   oc start-build  movies-pipeline
