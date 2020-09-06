pipeline {
    agent any
   
    options {
        disableConcurrentBuilds()
    }
   
      environment {
        Nuget_Proxy = "https://api.nuget.org/v3/index.json"
        scannerHome = tool name: 'sonar_scanner_dotnet',type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'

    }
   
    stages {       
        stage('nuget restore') {
            steps {   
                bat"dotnet clean"
                bat "dotnet restore"
             
            }
        }
       
     stage('Sonar analysis begin') {
        steps {
            withSonarQubeEnv('Test_Sonar') {
            bat "dotnet ${scannerHome}/SonarScanner.MsBuild.dll begin /k:\"sqs:NAGP-Assignment\"  /n:\"sqs:NAGP-Assignment\" /v:\"1.0.0\"  "
            
            }
        }
    }

      stage('code build') {
            steps {   
                bat " dotnet build -c Release"
            }
        }

    stage('Sonar analysis end') {
        steps { 
            withSonarQubeEnv('Test_Sonar') {
             bat "dotnet ${scannerHome}/SonarScanner.MsBuild.dll end"
            } 
        }
    }
          stage('docker image build') {
            steps { 
                bat "docker build -t hello-kube-ajay ."
            }
       
        }

        stage('Docker config'){
            parallel{
          stage('docker container precheck') {
              
              steps {
                  script{
                containerID = powershell(returnStdout: true, script:'docker ps -af name=c-hello-kube-ajay --format "{{.ID}}"')
                  if(containerID)
				    {
                        bat "docker stop ${containerID}"
					    bat "docker rm -f ${containerID}"
				     }
                  }
              }
          }
          
            stage('push docker image to dtr') {
               
                steps {   
                            bat "docker tag hello-kube-ajay dtr.nagarro.com:443/hello-kube-ajay"
                            bat "docker push dtr.nagarro.com:443/hello-kube-ajay"
                        }
            }
            }
        }
         stage('docker deployment') {
                steps {   
                            bat "docker run -d -p 8081:80 --name c-hello-kube-ajay dtr.nagarro.com:443/hello-kube-ajay"
                        }
            }
         
          stage('Helm chart Deployment') {
               
                steps {   

                    bat "helm install nagp-assignment-chart ./nagp-assignment-chart"
         
                      }
            }

        

    }
}
