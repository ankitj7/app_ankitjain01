pipeline {
  agent any

  environment {
    scannerHome = 'sonar_scanner_dotnet'
    username = 'ankitjain01'
    appName = 'nagp-devops-us'
  }

  options {
    timestamps()

    timeout(time: 1, unit: 'HOURS')

    buildDiscarder(logRotator(
      numToKeepStr: '3',
      daysToKeepStr: '10'
    ))
  }

  stages {

    stage("Nuget restore") {
      steps {
        echo "Nuget restore step started for ${BRANCH_NAME} branch"
        bat "dotnet restore"
      }
    }

    stage("Start Sonarqube analysis") {
      when {
        branch "master"
      }
      steps {
        echo "Start Sonarqube analysis step started"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll begin /k:sonar-${userName} /n:sonar-${userName} /v:1.0"
        }
      }
    }

    stage("Code build") {
      steps {
        echo "Code build step started"
        echo "Clean Old Build"
        bat "dotnet clean"

        // Buildthe project and its dependencies
        echo "Code Build"
        bat 'dotnet build -c Release -o "nagp-devops-us/app/build"'
        bat 'dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=nagp-devops-us.xml'
      }
    }

    stage("Test Case Execution") {
      when {
        branch "master"
      }
      steps {
        echo "Test Case Execution started"
      }
    }

    stage("Stop Sonarqube analysis") {
      when {
        branch "master"
      }
      steps {
        echo "Stop Sonarqube analysis step started"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll end"
        }
      }
    }

    stage("Release artifact") {
      when {
        branch "develop"
      }
      steps {
        echo "Release artifact step started"
        bat "dotnet publish -c Release -o ${appName}/app/${userName}"
      }
    }

    stage('Kubernetes Deployment') {
      steps {
        echo "Kubernetes Deployment step started"
        bat "kubectl apply -f deployment.yaml"
      }
    }
  }
}
