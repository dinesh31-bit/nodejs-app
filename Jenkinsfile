@Library('JenkinsSharedLib') _
pipeline{
    agent any
    tools{
        nodejs 'NodeJS 24.14.0'
    }
    triggers {
        githubPush()
    }
    stages{
        stage("git clone"){
            steps{
                git branch: 'main', credentialsId: 'dinesh31bitgit', url: 'https://github.com/dinesh31-bit/nodejs-app.git'
            }
        }
        stage("install packages to verify weather it is working good or not"){
            steps{
                sh "npm install"
            }
        }
        stage("ssh agent"){
            environment{
                nodejs_ip = '13.127.169.211'
            }
            steps{
            sshagent(['nodejs-ssh']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@${nodejs_ip} "mkdir -p /home/ec2-user/nodejs-app/ || true"
                ssh -o StrictHostKeyChecking=no ec2-user@${nodejs_ip} "rm -rf /home/ec2-user/nodejs-app/*"
                rsync -avz --exclude=node_modules --exclude=.git ./ ec2-user@${nodejs_ip}:/home/ec2-user/nodejs-app/
                '''
                }
            }
        }
        stage("install packages"){
            environment{
                nodejs_ip = '13.127.169.211'
            }
            steps{
                sshagent(['nodejs-ssh']) {
                    sh'''
                    ssh -o StrictHostKeyChecking=no ec2-user@${nodejs_ip} "cd /home/ec2-user/nodejs-app && 
                    npm install  && 
                    npm install -g pm2 && 
                    pm2 start app.js || pm2 restart app.js"
                    '''
                }
            }
        }
    }
    post{
        always{
            sh "echo 'it will run always'"
        }
        success{
            SendEmailNotification('success','dineshreddypala31@gmail.com')
            SendSlackNotification('devops-notification', 'SUCCESS')
        }
        failure{
            SendEmailNotification('Failure','dineshreddypala31@gmail.com')
            SendSlackNotification('devops-notification', 'FAILURE')
        }
    }
}
