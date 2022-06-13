pipeline {
  agent any
  environment{
    ServidorMaster="ServidorSSH"
    ServidorDeploy="192.168.0.21"
    PathDeploy="/home/deploy"
  }
  stages {
    stage('Test'){
      steps{
        sh 'mvn test'
      }
    }
    stage('Build') {
      steps {
        sh 'mvn package'
      }
    }
    stage('Entrega de Artefactos') {
      steps {
        archiveArtifacts 'target/jsf-text.war'
      }
    }
    // stage('SonarQube Analysis'){
    //   steps{
    //     withSonarQubeEnv(installationName: 'Sonarqube', credentialsId: 'token-sonarqube') {
    //       sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=root_istea-virtualizacion-local_AYCnNmcjtOtEzgO2dl-H'
    //     }
    //   }
    // }
    // stage('Quality Gate'){
    //   steps{
    //     timeout(time: 1, unit: 'MINUTES') {
    //       waitForQualityGate abortPipeline: true, credentialsId: 'token-sonarqube'
    //     }
    //   }
    // }

    stage('Delivery to artifact in server'){
      steps{
        //sshPublisher(publishers: [sshPublisherDesc(configName: 'ServidorSSH', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/to_implement', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: true, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/war', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/config', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.config')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      }
    }

    stage('Deploy to server'){
      steps{
        script{
          def remote = [:]
          remote.name = 'dev'
          remote.host = "${env.ServidorDeploy}"
          remote.allowAnyHosts = true
          withCredentials([sshUserPrivateKey(credentialsId: 'sshUser-key', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
            remote.user = userName
            remote.identityFile = identity
            //DEPLOY TO WAR
            writeFile file: 'deploy.sh', text: "cp /home/to_implement/*.war ${env.PathDeploy} \n sleep 1m"
            sshScript remote: remote, script: 'deploy.sh'
          }
        }
      }
    }


  }
}