pipeline {
    agent any

    environment {
        NODEJS_HOME = tool 'NodeJS'
        PATH = "${NODEJS_HOME}/bin:${PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[url: 'https://github.com/Bodiok007/TerraformAppForJenkins.git']]
                ])
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    npm install
                    npm run build
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/',
                    onlyIfSuccessful: true
                }
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    npm test
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'devops-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'USER')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY $USER@$REMOTE_HOST "mkdir -p $DEST_FOLDER"
                        scp -o StrictHostKeyChecking=no -i $SSH_KEY -r build/ package.json $USER@$REMOTE_HOST:$DEST_FOLDER
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY $USER@$REMOTE_HOST <<EOF
                          cd $DEST_FOLDER
                          if pm2 pid build/index.js > /dev/null; then
                            pm2 stop build/index.js
                          fi
                            npm install
                            pm2 start build/index.js
                          exit
                        EOF
                    '''
                }
            }
        }
    }
}