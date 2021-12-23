pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }

        stage('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }

        stage('Sonar Analysis'){
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=819de48f677879d4e90027ee77c6db9a50096a63 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }

        stage('Quality Gate'){
            steps{
                sleep(10)
                timeout(time: 1, unit: 'MINUTES'){
                waitForQualityGate abortPipeline:true
                }
                
            }
        }

        stage('Deploy Backend'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }

        stage('API Test'){
            steps{
                dir('api-test') {
                git credentialsId: 'github_login', url: 'git@github.com:andrezasecon/tasks-api-test.git' 
                 bat 'mvn test'          
                }
            }
        }

        stage('Deploy Frontend'){
            steps{
                dir('frontend') {
                git credentialsId: 'github_login', url: 'git@github.com:andrezasecon/tasks-frontend.git' 
                bat 'mvn clean package' 
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
               }
            }
        }

        stage('Functional Test'){
            steps{
                dir('functional-test') {
                git credentialsId: 'github_login', url: 'git@github.com:andrezasecon/tasks-funcional-tests.git' 
                bat 'mvn test'          
                }
            }
        }

        stage('Deploy Prod'){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }

        stage('Health Check'){
            steps{
                sleep(10)
                dir('functional-test') {
                bat 'mvn verify -Dskip.surefire.tests'                
                }
            }
        }
    }

    post{
        always{
            junit allowEmptyResults: true, testResults: 
            'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }

        unsuccessful{
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build $BUILD_NUMBER has failed', to: 'andrezasecon+jenkins@gmail.com'
        }

        fixed{
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build is fine!!!', to: 'andrezasecon+jenkins@gmail.com'
        }
    }
}


