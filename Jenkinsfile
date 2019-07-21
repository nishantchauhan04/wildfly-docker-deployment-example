def project = 'nishantchauhan'
def appName = 'ngv-poc-test'
def imageTag = "${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

pipeline {
  environment {
    registry = "nishantchauhan/testpipeline"
    registryCredential = 'dockerhub'
  }
  agent {
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: wildfly
    image: jboss/wildfly
    command:
    - cat
    tty: true
  - name: dind
    image: docker:1.11
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
}
  }
  stages {
    stage('Test') {
      steps {
        container('wildfly') {
          sh """
            sleep 5
          """
        }
      }
    }
    stage('Build and push image with Container Builder') {
    steps {
        container('dind') {
          sh """
            docker build -t ${imageTag} .
            docker login -u ${env.udocker} -p ${env.pdocker}
            docker push ${imageTag}
          """  
        }
      }
    }
    stage('Deploy Canary') {
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
          sh """
            sed -i.bak 's#richab123/app_server:0.0.1#${imageTag}#' ./k8s/*.yaml
            kubectl --namespace=default apply -f k8s/
          """
        } 
      }
    }
  }
}
