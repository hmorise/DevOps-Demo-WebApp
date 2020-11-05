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
        
        stage ('Deploy to test') {
            steps {
                echo 'Deploying the war file to tomcat'
                deploy adapters: [tomcat8(url: 'http://40.83.138.63:8080/', credentialsId: 'tomcat')], war: '**/*.war', contextPath: '/QAWebapp'
            }
        }
        
        stage('Store the Artifacts') {
            steps {
                script{
                    // Jenkins Configuration, Artifactory instance name
                    def server = Artifactory.server "Artifactory"
                    def rtMaven = Artifactory.newMavenBuild()
                    def buildInfo = Artifactory.newBuildInfo()
                
                        // Jenkins configuration Tool Name
                        rtMaven.tool = "Maven 3.6.3"

                        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
                        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
                        rtMaven.deployer.deployArtifacts = false
                    
                
                        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -e -Dmaven.repo.local=.m2', buildInfo: buildInfo
                
                        server.publishBuildInfo buildInfo
                }
            }
        }  
        
        stage('UI test') {
            steps {
                sh 'mvn -f functionaltest/pom.xml test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
            }
        }     
        
        //TODO: Performance Test Blazemeter step
        
        stage ('Prod Deploy') {
            steps {
                echo 'Deploying the war file to Prod'
                deploy adapters: [tomcat8(url: 'http://52.230.107.151:8080/', credentialsId: 'tomcat')], war: '**/*.war', contextPath: '/ProdWebapp'
            }
        }
        
        stage('Sanity test') {
            steps {
                sh 'mvn -f Acceptancetest/pom.xml test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
            }
        }    
        
        

    }
}
