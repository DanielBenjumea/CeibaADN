pipeline {
  //Donde se va a ejecutar el Pipeline
  agent {
    label 'Slave_Induccion'
  }

  //Opciones espec�ficas de Pipeline dentro del Pipeline
  options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    disableConcurrentBuilds()
  }

  //Una secci�n que define las herramientas �preinstaladas� en Jenkins
  tools {
    jdk 'JDK8_Centos' //Preinstalada en la Configuraci�n del Master
    gradle 'Gradle4.5_Centos' //Preinstalada en la Configuraci�n del Master
  }

  //Aqu� comienzan los �items� del Pipeline
  stages {
    stage('Checkout') {
      steps {
        echo '------------>Checkout<------------'
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/master']],
          doGenerateSubmoduleConfigurations: false,
          extensions: [],
          gitTool: 'Default',
          submoduleCfg: [],
          userRemoteConfigs: [[
            credentialsId: 'GitHub_DanielBenjumea',
            url:'https://github.com/DanielBenjumea/CeibaADN'
          ]]
        ])
      }
    }

    stage('Gradle permission') {
      steps {
        sh "chmod +x ./gradlew"
      }
    }

    stage('Compile & Unit Tests') {
      steps {
        echo '------------>Unit Tests<------------'
        sh "./gradlew --b ./build.gradle clean compileJava"
        sh "./gradlew --b ./build.gradle test"
      }
    }

    stage('Static Code Analysis') {
      steps {
        echo '------------>An�lisis de c�digo est�tico<------------'
        withSonarQubeEnv('Sonar') {
          sh "${tool name: 'SonarScanner', type:'hudson.plugins.sonar.SonarRunnerInstallation'}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
        }
      }
    }

    stage('Build') {
      steps {
        echo '------------>Build<------------'
        sh "./gradlew --b ./build.gradle build -x test"
      }
    }
  }

  post {
    always {
      echo 'This will always run'
    }
    success {
      echo 'This will run only if successful'
      junit "build/test-results/test/*.xml"
    }
    failure {
      echo 'This will run only if failed'
      mail (to: 'daniel.benjumea@ceiba.com.co', subject: "Failed Pipeline:${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")
    }
    unstable {
      echo 'This will run only if the run was marked as unstable'
    }
    changed {
      echo 'This will run only if the state of the Pipeline has changed'
      echo 'For example, if the Pipeline was previously failing but is now successful'
    }
  }
}
