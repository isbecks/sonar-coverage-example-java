/*
pipeline {
    agent any
    tools {
        jdk 'jdk10'
        maven 'M3'
    }

    environment {
        JAVA_HOME = "${jdk}"
    }

    stages {
        stage('Prepare') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'mvn install'
            }
        }

        stage('QA') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        def scannerHome = tool 'sonarqube-scanner'
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }
}
*/
node('java-slave') {

  //clone repo step from specific git URL
 // stage('SCM') {
 //   git 'https://github.com/isbecks/sonar-coverage-example-java'

    checkout([
        $class: 'GitSCM',
        branches: scm.branches,
        extensions: scm.extensions + [[$class: 'CleanCheckout']],
        userRemoteConfigs: scm.userRemoteConfigs
    ])
  //}

  

  //clean workspace and compile the source code
  stage("build") {
    sh "mvn clean compile"
  }

  // unit test job
  stage("unit-test") {
    sh "mvn test"
  }

  //run the sonar analisis using defined tool(as it is in the global tool configuration) and sonarEnv(as it is in the jenkins system configuration) as argument 
  stage('SonarQube analysis') {
    // requires SonarQube Scanner 2.8+
    def scannerHome = tool 'sonar-scanner';
    withSonarQubeEnv('sonar-server') {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }

  //perform quality gate step setting the time and retrieving the status from sonar webhook and then pass the QG status
  stage("Quality Gate"){
    
    timeout(time: 5, unit: 'MINUTES') {
      def qg = waitForQualityGate()
      if ((qg.status != 'WARN')&&(qg.status != 'OK')) {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
      }
    }
  }
}
