pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "youssefabu/vproappdock"
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

        stage('Build App Image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }

        stage('Upload Image'){
            steps{
                script {
                    docker.withRegistry('', registryCredential){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push("latest")

                    }
                }
            }
        }

        stage('Remove Unused docker Image'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }

        }

        stage('Kubernetes Deploy'){
            agent {label 'KOPS'}
                steps{
                    sh "helm install  --namespace prod vprofile-stack helm/vprofiecharts --set appimage=youssefabu/vproappdock"
                }
        }


    }


}