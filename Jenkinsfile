@@Library('jenkins-shared-library') _

def configMap = [
    project : "roboshop",
    component: "catalogue"
]

if( !"main".equalsIgnoreCase(env.BRANCH_NAME) ){
    nodejsEKSPipeline(configMap)
}
else{
    echo "Please proceed with PROD process"
}