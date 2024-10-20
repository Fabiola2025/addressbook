pipeline {
    agent { node { label "maven-sonarqube-node" } }   
    parameters {
      choice(name: 'aws_account',choices: ['999568710647', '637423546337', '922266408974','576900672829'], description: 'aws account hosting image registry')
      choice(name: 'Environment', choices: ['Dev', 'QA', 'UAT', 'Prod'], description: 'Target environment for deployment')
      string(name: 'ecr_tag', defaultValue: '1.7.0', description: 'Assign the ECR tag version for the build')
    }

    tools {
      maven "maven-3.9.8"
    }

    stages {
    stage('1. Git Checkout') {
      steps {
        git branch: 'release', credentialsId: 'Github-pat', url: 'https://github.com/Fabiola2025/addressbook.git'
      }
    }
    stage('2. Build with Maven') { 
      steps {
        sh "mvn clean package"
      }
    }
    stage('3. SonarQube Analysis') {
          environment {
                scannerHome = tool 'SonarQube-Scanner-6.2.1'
            }
            steps {
              withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                      sh """
                      ${scannerHome}/bin/sonar-scanner  \
                      -Dsonar.projectKey=addressbook \
                      -Dsonar.projectName='addressbook' \
                      -Dsonar.host.url=http://34.211.48.173:9000 \
                      -Dsonar.token=${SONAR_TOKEN} \
                      -Dsonar.sources=src/main/java/ \
                      -Dsonar.java.binaries=target/classes \
                     """
                  }
              }
        }
   stage('4. Docker Image Build') {
      steps {
          sh "aws ecr get-login-password --region us-west-2 | sudo docker login --username AWS --password-stdin ${aws_account}.dkr.ecr.us-west-2.amazonaws.com"
          sh "sudo docker build -t addressbook ."
          sh "sudo docker tag addressbook:latest ${aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
          sh "sudo docker push ${aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
      }
    }

    stage('5. Application Deployment in EKS') {
      steps {
       withKubeCredentials(caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '') {
          sh "kubectl apply -f manifest"
        }
      }
    }


    stages {
      stage('6. Monitoring Solution Deployment in EKS') {
        steps {
         withKubeConfig([credentialsId: 'kubeconfig']) {
            // Apply Kubernetes manifests for monitoring
               sh "kubectl apply -k monitoring"
                    
            // Install Helm
               sh "script/install_helm.sh"
                    
             // Install Prometheus
                sh "script/install_prometheus.sh"
         }
        }
     }
    }

    stage('7. Email Notification') {
      steps {
        mail bcc: 'nsuhfabiola@gmail.com', body: '''Build is Over. Check the application using the URL below:
         https://abook.dominionsystem.com/addressbook-1.0
         Let me know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', 
         subject: 'Application was Successfully Deployed!!', to: 'nsuhfabiola2gmail.com , kelvincaulker508@gmail.com'
      }
    }
  }
}





