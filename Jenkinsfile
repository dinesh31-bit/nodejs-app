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
            emailnotification('success','dineshreddypala31@gmail.com')
            Slacknotification('devops-notification', 'SUCCESS')
        }
        failure{
            emailnotification('Failure','dineshreddypala31@gmail.com')
            Slacknotification('devops-notification', 'FAILURE')
        }
    }
}
def emailnotification(String buildStatus,String reciver_email){

    def body = """
    <html>
    <body style="font-family: Arial, sans-serif; background-color: #f4f4f4; padding: 20px;">
        <h2 style="color: #2d87f0;">Jenkins Build Notification</h2>
        <p><strong>Build Result:</strong> ${buildStatus}</p>
        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
        <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
    </body>
    </html>
    """

    def subject = "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build ${buildStatus}"

    emailext(
        subject: subject,
        body: body,
        mimeType: 'text/html',
        to: reciver_email
    )
}
 def Slacknotification(String salckChannelName,String buildStatus){
	  
	    def color = buildStatus == 'SUCCESS' ? 'good' : (buildStatus == 'UNSTABLE' ? 'warning' : 'danger')
	    def message = "*${buildStatus}*: Job `${env.JOB_NAME}` - Build #${env.BUILD_NUMBER} \n" +
	                    "Details: ${env.BUILD_URL}console"
	      
          slackSend(channel: salckChannelName, color: color, message: message) 
	  
}