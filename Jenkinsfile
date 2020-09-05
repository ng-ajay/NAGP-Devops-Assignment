pipeline {
    agent any
   
    options {
        disableConcurrentBuilds()
    }
   
      environment {
        Nuget_Proxy = "https://api.nuget.org/v3/index.json"
     //   Scan_path = "C:/Users/ajaysrivastava/.dotnet/tools/dotnet-sonarscanner"
        scannerHome = tool name: 'sonar_scanner_dotnet',type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'
      //  home ='dotnet "C:/Program Files (x86)/Jenkins/tools/hudson.plugins.sonar.MsBuildSQRunnerInstallation/sonar_scanner_dotnet/SonarScanner.MsBuild.dll" begin /k:"sqs:NAGP-Assignment" /n:"sqs:NAGP-Assignment" /v:"1.0.0"'
      //  homeEnd ='dotnet "C:/Program Files (x86)/Jenkins/tools/hudson.plugins.sonar.MsBuildSQRunnerInstallation/sonar_scanner_dotnet/SonarScanner.MsBuild.dll" end'
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

        stage('Parallel jobs for docker'){
            parallel{
          stage('docker container precheck') {
              
              steps {
                  script{
                containerID = powershell(returnStdout: true, script:'docker ps --filter name=c-hello-kube-ajay --format "{{.ID}}"')
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
          
          stage('deploy to kubernetes cluster') {
               
                steps {   
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kube', namespace: '', serverUrl: 'https://kubernetes.docker.internal:6443') {

                    bat " helm install -f ./values.yaml fourthhelm ./my-chart"
                   //  bat "helm version"
                        
                    }
                           
                      }
            }
            stage('docker image run') {
                steps {   
                            bat "docker run -d -p 8081:80 --name c-hello-kube-ajay dtr.nagarro.com:443/hello-kube-ajay"
                        }
            }

        

    }
}
