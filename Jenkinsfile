pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "ivesaid53/vproappdock"
        registryCredential = "dockerhub"
    }

    stages{
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker App Image'){
        steps {
            script {
            dockerImage = docker.build registry + ":$BUILD_NUMBER" // imported from docker pipepline plugin //
            }
        }
        }
        stage('Upload Image to registry'){
        steps{
            script{ // use of script because we need to execute plugin based commands //
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER") // Build Number is a jenkins inbuild parameter which signifies the current build number //
                dockerImage.push('latest')
            }
            }
        }
        }
        stage('Remove Unused docker image') {
                  steps{
                    sh "docker rmi $registry:$BUILD_NUMBER" // Use of sh is because of actually using a shell command //  
                  }
        }

        stage('Kubernetes Deploy') {
        	  agent { label 'KOPS' }
                    steps {
                            sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"
                    }
        }

    }


}
