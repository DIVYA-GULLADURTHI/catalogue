pipeline {
    agent {
        label 'AGENT-1'
    }
    environment {
        appversion = ''
        REGION = "us-east-1"
        ACC_ID = "485067906741"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    } 
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    } 
   // Build  
    stages {
        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appversion = packageJson.version
                    echo "Package version: ${appversion}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                   sh """
                        npm install
                   """
                }
            }
        } 
        stage('Unit Testing') {
            steps {
                script {
                   sh """
                        echo "unit tests"
                   """
                }
            }
        } 
        stage('Sonar Scan') {
            environment {
                scannerHome = tool 'sonar-7.2'
            }
            steps {
                script {
                   // Sonar Server envrionment
                   withSonarQubeEnv(installationName: 'sonar-7.2') {
                         sh "${scannerHome}/bin/sonar-scanner"
                   }
                }
            }
        } 
        stage('Docker Build') {
            steps {
                script {
                    withAWS(credentials: 'aws-credits', region: 'us-east-1') {
                        sh """
                            aws ecr describe-repositories \
                            --repository-names ${PROJECT}/${COMPONENT} \
                            --region ${REGION} || \
                            aws ecr create-repository \
                            --repository-name ${PROJECT}/${COMPONENT} \
                            --region ${REGION}

                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appversion} . 
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appversion}
                        """
                    }
                }
            }
        } 
        stage('Trigger Deploy') { 
            when{
               expression { params.deploy }
            }
            steps {
                script {
                   build job: 'catalogue-cd',
                    parameters: [
                        string(name: 'appversion', value: "${appversion}"),
                        string(name: 'deploy_to', value: 'dev')
                    ],
                   propagate: false, // even sg fails VPC will not be effected
                   wait: false // VPC will not wait for SG pipeline completion 
                }
            }
        } 
    } 
    
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        } 
        success { 
            echo 'Hello Success'
        } 
        failure { 
            echo 'Hello failure'
        }
    }

}


 