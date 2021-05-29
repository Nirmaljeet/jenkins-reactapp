pipeline {
    agent any

    stages {
            stage('Git clone') {
            steps {
                sh "git clone https://github.com/Nirmaljeet/jenkins-reactapp.git"
            }
        }
        
        stage('Image Build') {
            steps {
                sh "cd jenkins-reactapp && docker build -t react-prod:$BUILD_NUMBER ."
            }
        }
        stage('Push Image') {
            steps {
                withAWS(credentials:'AWS'){
                sh "aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/n7v0q5y9"
                sh "docker tag react-prod:$BUILD_NUMBER public.ecr.aws/n7v0q5y9/react-prod:$BUILD_NUMBER "
                sh "docker push public.ecr.aws/n7v0q5y9/react-prod:$BUILD_NUMBER"
                    
                }
            }
        }
        
        

        stage('Creating Infra') {
            steps {
                withAWS(credentials:'AWS'){
                    sh "cd jenkins-reactapp/ && aws cloudformation update-stack --stack-name react-cluster --template-body file://ecs-template.yml --parameters ParameterKey=DesiredCount,ParameterValue=2 ParameterKey=DesiredCapacity,ParameterValue=2 --capabilities CAPABILITY_IAM --region us-east-1"
                    
                }
            }
        }


        stage('Waiting for stack to complete') {
            steps {
                withAWS(credentials:'AWS'){
                    sh 'aws cloudformation describe-stacks --stack-name react-cluster --region us-east-1'
                }
            }
        }
       
       stage('Cleanup') {
            steps {
                sh "rm -rf jenkins-reactapp/"
            }
        }
        
    }
}
