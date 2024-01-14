pipeline {
  agent any
  tools {
  maven 'Maven'
  }
    
	stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/efeben007/devops-clone-repo.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }   
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }		
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://3.227.46.127:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "project-a-libs-release-local",
                    snapshotRepo: "project-a-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog", // credential ID from Jenkins global credentials
                    releaseRepo: "project-a-libs-release-local",
                    snapshotRepo: "project-a-libs-snapshot-local"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "Maven", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Staging Server') {
            
            steps {
                  sshagent(['ssh_agent']) {
                       sh "chmod 400  efebenkey.pem"
                       sh "ls -lah"
                        sh "scp -i efebenkey.pem -o StrictHostKeyChecking=no dockerfile ubuntu@34.204.155.78:/home/ubuntu"
                        sh "scp -i efebenkey.pem -o StrictHostKeyChecking=no dockerfile ubuntu@34.204.155.78:/home/ubuntu"
                        sh "scp -i efebenkey.pem -o StrictHostKeyChecking=no push-dockerhub.yaml ubuntu@34.204.155.78:/home/ubuntu"
                    }
                }
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i efebenkey.pem -o StrictHostKeyChecking=no ubuntu@34.204.155.78 -C \"ansible-playbook  -vvv -e build_number=${BUILD_NUMBER} push-dockerhub.yaml\""       
                    }
                }
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i efebenkey.pem -o StrictHostKeyChecking=no deploy_service.yaml ubuntu@18.210.253.218:/home/ubuntu"
                        }
                }
        } 

    stage('Waiting for Approvals') {
            
        steps{
		input('Test Completed ? Please provide  Approvals for Prod Release ?')
			 }
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        //sh "ssh -i efebenkey.pem -o StrictHostKeyChecking=no ubuntu@18.210.253.218 -C \"kubectl set image deployment/ranty customcontainer=efeben007/efe-repo:${BUILD_NUMBER}\""
                        //sh "ssh -i efebenkey.pem -o StrictHostKeyChecking=no ubuntu@18.210.253.218 -C \"kubectl delete deployment ranty && kubectl delete service ranty\""
                        sh "ssh -i efebenkey.pem -o StrictHostKeyChecking=no ubuntu@18.210.253.218 -C \"kubectl apply -f deploy_service.yaml\""
                        //sh "ssh -i efebenkey.pem -o StrictHostKeyChecking=no ubuntu@18.210.253.218 -C \"kubectl apply -f service.yaml\""
                    }
                }  
        } 
   } 
}



