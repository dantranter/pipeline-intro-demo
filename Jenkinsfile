pipeline {
  agent {
    label 'java-maven'
  }
  stages {
    stage('Buzz Build') {
      steps {
        container('global-maven3-jdk8') {
          echo 'Buzz Build'
          sh '''echo I am a $BUZZ_NAME
          ./jenkins/build.sh
          '''
          script {
                    // Obtain an Artifactory server instance, defined in Jenkins credentials:
                    def server = Artifactory.newServer url: 'https://devops-demo.app/artifactory/webapp/#/home', credentialsId: 'JFrog_Cisco'

                    // Read the download and upload specs:
                    def downloadSpec = readFile 'resources/props-download.json'
                    def uploadSpec = readFile 'resources/props-upload.json'

                    // Download files from Artifactory:
                    def buildInfo1 = server.download spec: downloadSpec
                    // Upload files to Artifactory:
                    def buildInfo2 = server.upload spec: uploadSpec

                    // Merge the local download and upload build-info instances:
                    buildInfo1.append buildInfo2

                    // Publish the merged build-info to Artifactory
                    server.publishBuildInfo buildInfo1
                }
           archiveArtifacts(artifacts: 'target/*.jar', fingerprint: true)
        }
      }
    }
    stage('Buzz Test') {
      parallel {
        stage('Testing A') {
          steps {
            container('global-maven3-jdk8') {
              echo 'Buzz Test A'
              sh './jenkins/test-all.sh'
              junit '**/surefire-reports/**/*.xml'
            }
          }
        }
        stage('Testing B') {
          steps {
            echo 'Buzz Test B'
            sh '''sleep 10
            echo done.'''
          }
        }
      }
    }
  }
  environment {
    BUZZ_NAME = 'Worker Bee'
  }
}
