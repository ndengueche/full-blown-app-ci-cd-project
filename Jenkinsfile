def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    environment{
        WORKSPACE = "${env.WORKSPACE}"
    }
    
    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the app code'
                git branch: 'main', url: 'https://github.com/ndengueche/full-blown-app-ci-cd-project.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Archiving...'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
            
        }
        
        stage('Unit Test'){
            steps {
                sh 'mvn test'
            }
        }
  
        stage('Integration Test'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
   
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
           
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        
        stage ('SonarQube scanning'){
            steps {
                
                withSonarQubeEnv('SonarQube') {
                    
                sh """
                mvn sonar:sonar \
                  -Dsonar.projectKey=JavaWebApp \
                  -Dsonar.host.url=http://172.31.51.223:9000 \
                  -Dsonar.login=d6e37a7391ce199a0d8d7c0cfcd496698dc586c0
                  
                 """
                }
            }
        }
        
        stage('Quality Gate'){
            steps{
                waitForQualityGate abortPipeline: true
            }
            
        }
        
        stage('Upload Artifact to Nexus'){
            steps{
                sh 'mvn clean deploy -DskipTests'
            }
            
        }
        
        stage('Deploy to DEV') {
            environment {
                HOSTS = "dev"
            }
            steps {
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
            }
        }
        
        stage('Deploy to Stage') {
            environment {
                HOSTS = "stage"
            }
            steps {
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
            }
        }
        
        stage('Approval') {
            steps {
                input('Do you want to proceed?')
            }
        }
        
        stage('Deploy to Prod') {
            environment {
                HOSTS = "prod"
            }
            steps {
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
            }
        }
        
    }
    
    post {
            always {
                echo 'Slack Notifications.'
                    slackSend channel: '#my-notification-channel', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            }
        }
    
}
    
