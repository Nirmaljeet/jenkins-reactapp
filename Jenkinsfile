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
                    sh "cd jenkins-reactapp/ && aws cloudformation update-stack --stack-name react-cluster --template-body file://ecs-template.yml --parameters ParameterKey=DesiredCount,ParameterValue=3 ParameterKey=DesiredCapacity,ParameterValue=3 --capabilities CAPABILITY_IAM --region us-east-1"
                    
                }
            }
        }
        
        stage('Waiting for stack to complete') {
            steps {
                withAWS(credentials:'AWS'){
                    sh 'aws cloudformation wait stack-update-complete  --stack-name "https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/stackinfo?stackId=arn%3Aaws%3Acloudformation%3Aus-east-1%3A196202912397%3Astack%2Freact-cluster%2Fa23b2dc0-ba41-11eb-9aba-0efa5da0ad91" --region us-east-1'
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
