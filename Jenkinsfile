pipeline{
    agent{
        label 'AGENT-1'
    }
    options {
        ansiColor('xterm')
    }
    environment{
        REGION="us-east-1"
        PROJECT="roboshop"
        COMPONENT="user"
    }
    parameters{
        string(name: 'appVersion', defaultValue: '', description: 'Please enter Image version')
        choice(name: 'deploy', choices: ['dev', 'qa', 'prod'], description: 'Pick Environment')

    }
    stages {
        stage('Deploy'){
            steps{
                withAWS(credentials: 'aws-creds', region: "${REGION}"){
                 script{
                    sh """
                    aws eks update-kubeconfig --name "${PROJECT}-${params.deploy}" --region ${REGION}
                    kubectl apply -f namespace.yaml
                    sed -i "s/IMAGEVERSION/${params.appVersion}/g" values-${params.deploy}.yaml
                    helm upgrade --install ${COMPONENT} -f values-${params.deploy}.yaml .
                    """
                  }
                }
               
            }
        }
        stage('check-status'){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: "${REGION}"){

                    def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/${component} --timeout=10s -n $PROJECT || echo FAILED").trim()
                    if (deploymentStatus.contains("successfully rolled out")) {
                        echo "Deployment is success"
                    }
                    else {
                        sh """
                            helm rollback $COMPONENT
                        """
                        def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/${component} --timeout=10s -n $PROJECT || echo FAILED").trim()
                        if (rollbackStatus.contains("successfully rolled out")) {
                            error "Deployment is Failure, Rollback Success"
                        }
                        else{
                            error "Deployment is Failure, Rollback Failure. Application is not running"
                        }
                    }

                }
              }
            }  
      }
    }
    post{
        always {
            echo "Pipeline exection completed"
            deleteDir()
        }
        success{
            echo "Success"
        }
        failure{
            echo "Failure"
        }
    }
 
}
