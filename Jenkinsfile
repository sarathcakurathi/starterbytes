pipeline {
	agent any

	tools {
	  // Install the Maven version configured as "M3" and add it to the path.
	  maven "maven3"
	}

	parameters {
		booleanParam(name: 'BUILD_DEPLOY_ARTIFACTS', defaultValue: 'true', description: 'Skip build and artifact deployment')
	}
	
	stages {
		stage('Set Parameters') {
			steps {
				script {
					if (env.BUILD_NUMBER.equals("1")) {
						currentBuild.displayName = 'Parameter loading'
						currentBuild.description = 'Please restart pipeline'
						currentBuild.result = 'ABORTED'
						error('Stopping initial build as we only want to get the parameters')
					}
				}
			}
			
		}
		
		stage('Build') {
			 steps {
				script {
					// Get some code from a GitHub repository
					//git 'https://github.com/sarathcakurathi/starterbytes.git'
					echo "Parameter:: ${BUILD_DEPLOY_ARTIFACTS}"
					def var = (params.BUILD_DEPLOY_ARTIFACTS == false)
					echo "Var: ${var}"
					if (params.BUILD_DEPLOY_ARTIFACTS == true) {
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
					if (params.BUILD_DEPLOY_ARTIFACTS == true) {
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