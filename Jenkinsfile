pipeline {
   agent any
   environment {
       registry = "afsal983/ilnaaccounting"
       GOCACHE = "/tmp"
       kubeApi = "https://192.168.0.201:6443"
       namespace = "app"
       application = "godb"

   }
   stages {
       stage('Build') {
           agent {
               docker {
                   image 'golang'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${GOPATH}/src'
               sh 'mkdir -p ${GOPATH}/src/hello-world'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
               // Build the app.
               sh 'go build'
           }
       }
       
	stage('Publish') {
           environment {
               registryCredential = 'dockerhub'
           }
           steps{
               script {
                   def appimage = docker.build registry + ":$BUILD_NUMBER"
                   docker.withRegistry( '', registryCredential ) {
                       appimage.push()
                       appimage.push('latest')
                   }
               }
           }
       }
stage('k8s Deployment') {

        steps {

            echo "Deploying godb to k8s cluster"

            script {

                withKubeConfig([credentialsId: kubeCredential, serverUrl: kubeApi]) {

                  env.deployTo.tokenize(",").each { ns ->

                    echo "Deploying application to namespace: app"

                    sh "kubectl get all -n app -l app=geodb"

                    sh "kubectl apply -f ./k8s -n app"

                    sh "sleep 5"

                    sh "kubectl get all -n app -l app=geodb"

                  }

                }

            }

            echo "Successfully deployed ${app} to k8s cluster"

        }

    }
   }
}
