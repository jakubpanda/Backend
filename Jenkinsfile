def imageName="jakubpanda/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def dockerTag=""

pipeline {
    agent {
        label 'agent'

    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }


    stages {
  stage('Get Code') {
    steps {
      checkout scm
    }
  }

  stage('show file') {
    steps {
      sh 'ls -a'
    }
  }

  stage('run unit') {
    steps {
      sh '''pip3 install -r requirements.txt
python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml

'''
    }
  }
 

  stage('analiza') {
    steps {
      withSonarQubeEnv('SonarQube') {
      sh "${scannerHome}/bin/sonar-scanner"
      }
      timeout(time: 1, unit: 'MINUTES') {
      waitForQualityGate abortPipeline: true
      }
    }
  }
  
    stage('Build application image') {
        steps {
            script {
            dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
            applicationImage = docker.build("$imageName:$dockerTag",".")
            docker.withRegistry("$dockerRegistry", "$registryCredentials") {
            applicationImage.push()
            applicationImage.push('latest')
            }

            }
          }
    }   



}

post {
  always {
    junit 'test-results/*.xml'
    cleanWs()
  }
}

}
