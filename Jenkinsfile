pipeline {
  agent any
  stages {
    stage('test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Publish to Nexus') {
      steps {
        script {
          pom = readMavenPom file: "pom.xml";
          // Find built artifact under target folder
          filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
          // Print some info from the artifact found
          echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
          // Extract the path from the File found
          artifactPath = filesByGlob[0].path;
          // Assign to a boolean response verifying If the artifact name exists
          artifactExists = fileExists artifactPath;

          if(artifactExists) {
            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

            nexusArtifactUploader(
              nexusVersion: NEXUS_VERSION,
              protocol: NEXUS_PROTOCOL,
              nexusUrl: NEXUS_URL,
              groupId: pom.groupId,
              version: pom.version,
              repository: NEXUS_REPOSITORY,
              credentialsId: NEXUS_CREDENTIAL_ID,
              artifacts: [
                // Artifact generated such as .jar, .ear and .war files.
                [artifactId: pom.artifactId,
                classifier: '',
                file: artifactPath,
                type: pom.packaging],

                // Lets upload the pom.xml file for additional information for Transitive dependencies
                [artifactId: pom.artifactId,
                classifier: '',
                file: "pom.xml",
                type: "pom"]
              ]
            );

          } 
          else {
            error "*** File: ${artifactPath}, could not be found";
          }
        }
      }
    }

    stage('build') {
      steps {
        sh 'mvn package'
      }
    }
    stage('Delivery to artifact in server'){
      steps{
        //sshPublisher(publishers: [sshPublisherDesc(configName: 'ServidorSSH', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/to_implement', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: true, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/war', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/|', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.config')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      }
    }
    environment {
      ServidorMaster = 'ServidorSSH'
      NEXUS_VERSION = 'nexus2'
      NEXUS_PROTOCOL = 'http'
      NEXUS_URL = '192.168.0.137:8081/nexus'
      NEXUS_REPOSITORY = 'releases'
      NEXUS_CREDENTIAL_ID = 'userNexus'
    }
  }
}