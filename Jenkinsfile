@Library('my-shared-library') _

pipeline{

    agent any
    //agent { label 'Demo' }

    parameters{

        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'rkiruthiga')
        string(name: 'ArtifactoryURL', description: "URL of the Artifactory server", defaultValue: '21:12:21:12:8082')
    }

    stages{
         
        stage('Git Checkout'){
                    when { expression {  params.action == 'create' } }
            steps{
            gitCheckout(
                branch: "main",
                url: "https://github.com/praveen1994dec/Java_app_3.0.git"
            )
            }
        }
         stage('Unit Test maven'){
         
         when { expression {  params.action == 'create' } }

            steps{
               script{
                   
                   mvnTest()
               }
            }
        }
         stage('Integration Test maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnIntegrationTest()
               }
            }
        }
        stage('Static code analysis: Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'sonarqube-api'
                   statiCodeAnalysis(SonarQubecredentialsId)
               }
            }
       }
       stage('Quality Gate Status Check : Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'sonarqube-api'
                   QualityGateStatus(SonarQubecredentialsId)
               }
            }
       }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnBuild()
               }
            }
        }
       // stage('Connect to Jfrog'){
         //   steps{
          //      script{
             //       rtServer(
                //        id: 'Artifactory-1',
                  //      url: 'http://65.2.35.73:8081/',
                    //        // If you're using username and password:
                      //  username: 'admin',
                        //password: 'Jfrog123@'
  //                  )    
    //            }
      //      }
        //}

       //   stage('Artifactory : Jfrog') {
         //   when { expression { params.action == 'create' } }
           // steps {
             //   script {
               //     sh 'curl -X PUT -u admin -T kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar http://65.2.35.73:8082/artifactory/libs-release-local/'
       //         }
         //   }
   //     }
        stage('Push to Jfrog'){
            when { expression { params.action == 'create' } }
            steps{
                script{
                    echo "Attempting to push the artifacts to jfrog Artifactory"
                    withCredentials([usernamePassword(
                        credentialsId: "Artifactory"
                        usernameVariable: "USER",
                        passwordVariable: "PASS"
                    )]) {
                        //Using Artifactory username and password variables
                        echo "Username: $USER"
                        echo "Password: $PASS"
                        
                        def curlCommand="curl -u '${USER}:${PASS} -T target/*.jar ${params.ArtifactoryURL}/artifactory/example-repo-local/"
                        echo "Executing Curl Command: $curlCommand"
                        sh curlCommand
                      }
                    }
                 }
            }    
        
        stage('Docker Image Build'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
         stage('Docker Image Scan: trivy '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
        stage('Docker Image Push : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }   
        stage('Docker Image Cleanup : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }      
    }
}
