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
                    bat "\"${scannerHome}/bin/sonar-scanner\" -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=819de48f677879d4e90027ee77c6db9a50096a63 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=exclusions=**./.mvn/**,**/src/test**,**/model/**,**Application.java"
                }
            }
        }

        stage('Quality Gate'){
            steps{
                sleep(300)
                timeout(time: 5, unit: 'MINUTES'){
                waitForQualityGate abortPipeline:true
                }
            }
        }
    }
}
