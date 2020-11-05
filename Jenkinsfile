pipeline {
    agent any
    
    
     tools { 
        maven 'Maven 3.6.3' 
        jdk 'jdk8' 
    }
    

    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
            
        }
        stage('Checkout') {
            steps {
                echo 'Checking out the code from github'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/hmorise/DevOps-Demo-WebApp.git']]])

            }
        }
         stage('Sonarqube') {
             environment {
                 scannerHome = tool 'sonarqube'
             }
            steps {
                echo 'Setting sonarqube stage'
                withSonarQubeEnv('sonarqube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('maven build') {
            steps {
                echo 'building...'
                sh 'mvn clean install'
            }
        }
        
        stage ('deploy to test') {
            steps {
                echo 'Deploying the war file to tomcat'
                deploy adapters: [tomcat8(url: 'http://40.83.138.63:8080/', credentialsId: 'tomcat')], war: '**/*.war', contextPath: '/QAWebapp'
            }
        }
        
        

    }
}
