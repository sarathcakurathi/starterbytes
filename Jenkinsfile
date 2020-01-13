pipeline {
	agent any

	tools {
	  // Install the Maven version configured as "M3" and add it to the path.
	  maven "maven3"
	}

	parameters {
		BooleanParameter(name: 'SKIP_ARTIFACT_BUILD_DEPLOY', defaultValue: 'true', description: 'Skip build and artifact deployment')
	}

	config = [
		skipSetParameters : true
	]
	
	stages {
		stage('Set Parameters') {
			when {
                showOnlyWhen {
                    expression { return !config.skipSetParameters } // evaluated after the stage is executed
                }
            }
			steps {
				currentBuild.displayName = 'Parameter loading'
				addBuildDescription('Please restart pipeline')
				currentBuild.result = 'ABORTED'
				error('Stopping initial manually triggered build as we only want to get the parameters')
			}
			
		}
		
		stage('Build') {
			 steps {
				script {
					// Get some code from a GitHub repository
					//git 'https://github.com/sarathcakurathi/starterbytes.git'
					echo "Parameter:: ${SKIP_ARTIFACT_BUILD_DEPLOY}"
					def var = (params.SKIP_ARTIFACT_BUILD_DEPLOY == false)
					echo "Var: ${var}"
					if (params.SKIP_ARTIFACT_BUILD_DEPLOY == false) {
						def server = Artifactory.server 'art'
						def rtMaven = Artifactory.newMavenBuild()
						rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
						rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
						def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
					}
				}      
			}
		}
		 
		stage('Deploy Artifacts') {
			steps {
				script {
					if (SKIP_ARTIFACT_BUILD_DEPLOY == false) {
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
        
    }
	  
	post {
		success {
		   //junit '**/target/surefire-reports/TEST-*.xml'
		   //archiveArtifacts 'target/*.jar'
		   echo "Done"
		}
	}
}