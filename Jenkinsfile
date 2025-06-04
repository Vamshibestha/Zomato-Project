pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'mysonar'
    }
    stages {
        stage ("Clean") {
            steps {
                cleanWs()
            }
        }
        stage ("Code") {
            steps {
                git branch: 'main', url: 'github url'
            }
        }
        stage("Sonarqube Analysis") {
            steps{
                withSonarQubeEnv('mysonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato 
\
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage ("Quality Gates") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage ("Install dependencies") {
            steps {
                sh 'npm install'
            }
        }
        stage ("OWASP") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --
 disableNodeAudit', odcInstallation: 'Dp-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Trivy scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage ("Build Dockerfile") {
            steps {
                sh 'docker build -t image1 .'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker tag image1 docker hub username/repo:imagetag"
                        sh "docker push docker hub username/repo:imagetag""
                    }
                }
            }
        }
        stage ("Scan image") {
            steps {
                sh 'trivy image docker hub username/repo:imagetag"'
            }
        }
        stage ("Deploy") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 docker hub 
username/repo:imagetag'
            }
        }
    }
 }
