pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    stages{
        stage ('clean workspace'){
            steps{
            cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jjaswanth66/amazon-eks-jenkins-terraform-aj7.git']])
            }
        }
        stage ('maven compile') {
            steps{
                sh 'mvn clean compile'
            }
        }
        stage ('sonarqube analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage ('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build war file'){
            steps{
                sh 'mvn clean install package'
            }
        }
        stage('build and push to docker hub'){
            steps{
                script{
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker',toolName:'docker') {
                        sh "docker build -t petclinic1 ."
                        sh "docker tag petclinic1 jagadabhijaswanth132/pet-clinic123:latest "
                        sh "docker push jagadabhijaswanth132/pet-clinic123:latest "
                    }
                }
            }
        }
        stage(trivy){
            steps{
                sh "trivy image jagadabhijaswanth132/pet-clinic123"
            }
        }
        stage('deploy to container'){
            steps{
                sh "docker run -d --name pet1 -p 8082:8080 jagadabhijaswanth132/pet-clinic123"
            }
        }
    }
}
