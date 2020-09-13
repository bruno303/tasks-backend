pipeline {
    agent any

    stages {
        stage('Build backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Unit tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=1e51ba5de6acf4dd539b4be6899b32c130f10e8a -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                sleep(30)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: '/tasks-backend', war: 'target/tasks-backend.war'
            }
        }

        stage('API Test') {
            steps {
                dir('api-test') {
                    git 'https://github.com/bruno303/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/bruno303/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: '/tasks', war: 'target/tasks.war'
                }
            }
        }

        stage('Functional Test') {
            steps {
                dir('functional-test') {
                    git 'https://github.com/bruno303/tasks-functional-tests'
                    sh 'mvn test'
                }
            }
        }
    }
}