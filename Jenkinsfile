pipeline {
 agent { node { label "maven-sonarqube-server" } }
 parameters   {
   choice(name: 'aws_account',choices: ['999568710647', '4568366404742', '922266408974','576900672829'], description: 'aws account hosting image registry')
   choice(name: 'Environment',choices: ['Dev', 'QA', 'UAT','Prod'], description: 'Target environment for deployment')
   string(name: 'ecr_tag',defaultValue: '1.0.0', description: 'Assign the ecr tag version for the build')
       }
tools {
    maven "maven-3.9.8"
    }
    stages {
      stage('1. Git Checkout') {
        steps {
          git branch: 'release', credentialsId: 'Github-pat', url: 'https://github.com/GroupAEKS/addressbook-app'
        }
      }
      stage('2. Build with maven') { 
        steps{
          sh "mvn  clean package"
         }
       }
      stage('3. SonarQube analysis') {
      environment {SONAR-TOKEN = credentials('sonar-token')}
      steps {
       script {
         def scannerHome = tool 'SonarQube-Scanner-6.2.1';
         withSonarQubeEnv("sonarqube-integration") {
         sh "${tool("SonarQube-Scanner-6.2.1")}/bin/sonar-scanner  \
           -Dsonar.projectKey=addressbook-application \
           -Dsonar.projectName='addressbook-application' \
           -Dsonar.host.url=http://54.201.142.70:9000 \
           -Dsonar.token=$sonar-token \
           -Dsonar.sources=src/main/java/ \
           -Dsonar.java.binaries=target/classes"
          }
         }
       }
      }
      stage('4. Docker image build') {
         steps{
          sh "aws ecr get-login-password --region us-west-2 | sudo docker login --username AWS --password-stdin ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com"
          sh "sudo docker build -t addressbook ."
          sh "sudo docker tag addressbook:latest ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
          sh "sudo docker push ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
         }
       }
      stage('5. Application deployment in eks') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'kubeconfig', serverUrl: '') {
          sh "kubectl apply -f manifest"
          }
         }
       }
      stage('6. Monitoring solution deployment in eks') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'kubeconfig', serverUrl: '') {
          sh "kubectl apply -f monitoring"
          }
         }
       }
      stage ('7. Email Notification') {
         steps{
         mail bcc: 'fusisoft@gmail.com', body: '''Build is Over. Check the application using the URL below. 
         https//abook.dominionsystem.com/addressbook-1.0
         Let me know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', cc: 'fusisoft@gmail.com', from: '', replyTo: '', subject: 'Application was Successfully Deployed!!', to: 'fusisoft@gmail.com'
      }
    }
 }
}



