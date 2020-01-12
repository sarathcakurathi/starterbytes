pipeline {
   agent any

   tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven3"
   }

   stages {
      stage('Build') {
         steps {
            script {
                // Get some code from a GitHub repository
                //git 'https://github.com/sarathcakurathi/starterbytes.git'
                def server = Artifactory.server 'art'
                def rtMaven = Artifactory.newMavenBuild()
                rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
                rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
                def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
            }
            
            Stage('Deploy Artifacts') {
                steps {
                    script {
                        try {
                            rtMaven.deployer.deployArtifacts buildInfo
                            server.publishBuildInfo buildInfo
                        } catch(Exception e) {
                            echo "Skipping artifact deployments, possibly due to artifacts already existing"
                        }
                    }
                }
            }
         }

         post {
            success {
               junit '**/target/surefire-reports/TEST-*.xml'
               archiveArtifacts 'target/*.jar'
            }
         }
      }
   }
}