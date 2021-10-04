pipeline {
   options {
            buildDiscarder(logRotator(numToKeepStr: "10"))
        }

    agent any
    
    environment {
        ECR_REPO = '243060292047.dkr.ecr.eu-west-1.amazonaws.com/tech-talk'
       teams_url = "https://majidalfuttaim.webhook.office.com/webhookb2/9c4d9eba-3f45-43eb-b60f-01487a9fc92e@b09ab65e-9063-4862-afa2-d75ab48bba74/JenkinsCI/2d1c5570cf2f4f638211201e331a823f/ea36fd4b-baff-4d76-b4f3-ddf2a3c0cf1d"
       ENVIRONMENT = "${env.BRANCH_NAME == 'develop' ? 'dev' : 'prod'}"

        }
      stages {
            
          stage('Build start notification') {
            steps {
                office365ConnectorSend (
                    color: '#223c6e',
                    message: "TechTalk:Started <br/>\nJob: '${env.JOB_NAME}' <br/>\nBuild: '[${env.BUILD_NUMBER}]' <br/>\nBranch: '${env.BRANCH_NAME}'",
                    status: 'Started',
                    webhookUrl: "${env.teams_url}"
                )       
            }
        } 
        
         stage('Commit Details') { 
            steps {
                    script {
                        env.pr_id =  sh (script: 'git log -n 1 --pretty=format:"%s" |cut -d "#" -f2 | cut -d" " -f1 |tr -dc "0-9"', returnStdout: true).trim()
                        env.pr_author = sh (script: 'git log -n 1 --pretty=format:"%an"', returnStdout: true).trim()
                    }                   
                }
        }
        
        stage('Build-Push Image') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'master'
                }
            }
            steps {
                sh script:'''
                       docker build --no-cache -t $ECR_REPO:$ENVIRONMENT-latest . --network host
                    '''
            } 

        }
        stage('Test') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'master'
                }
            }
            steps {
                sh script:'''
                       echo Do tests here :)
                    '''
            }
        }
        stage('Push') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'master'
                }
            }
            steps {
                sh script:'''
                    /usr/local/bin/aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin $ECR_REPO
                    docker push $ECR_REPO:$ENVIRONMENT-latest
                '''
            }
        } 

  post {
        always {
            office365ConnectorSend (
                color: "${currentBuild.result == 'SUCCESS' ? '#40a832' : '#FF0000'}",
                message: "Tech-Talk-Build:${currentBuild.result} for PR#'${env.pr_id}' <br/>\nPR Author: '${env.pr_author}' <br/>\nJob: '${env.JOB_NAME}' <br/>\nBuild: '[${env.BUILD_NUMBER}]' <br/>\nBranch: '${env.BRANCH_NAME}'",
                status: "${currentBuild.result}",
                webhookUrl: "${env.teams_url}"
            )
            cleanWs()
        }
    }  
        
}