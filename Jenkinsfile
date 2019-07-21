def project = 'ngv-poc'
def  appName = 'node'

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
  serviceAccountName: ngv-poc
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
            docker build -t nishantchauhan/hcl-test:${env.BUILD_NUMBER} .
          """  
        }
      }
    }
    stage('Deploy Canary') {
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
          sh("kubectl --namespace=production apply -f k8s/services/")
          sh("kubectl --namespace=production apply -f k8s/canary/")
          sh("echo http://`kubectl --namespace=production get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        } 
      }
    }
  }
}
