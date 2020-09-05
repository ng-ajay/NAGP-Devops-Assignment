pipeline {
    agent any
   
    options {
        disableConcurrentBuilds()
    }
   
      environment {
        Nuget_Proxy = "https://api.nuget.org/v3/index.json"
        Scan_path = "C:/Users/ajaysrivastava/.dotnet/tools/dotnet-sonarscanner"
    }
   
    stages {       
        stage('nuget restore') {
            steps {   
                bat"dotnet clean"
                bat "dotnet restore"
             
            }
        }
       
      
        stage('Unit Testing') {
            steps {
                 echo 'testing'    
            }
        }
   
   
          stage('code build') {
            steps {   
                bat " dotnet build -c Release -o  WebApplication4/app/build WebApplication4.sln"
            }
        }
          stage('docker image build') {
            steps {   
                bat "docker build -t i_ajaysrivastava_master -f Dockerfile ."
            }
        }

         
    }
}