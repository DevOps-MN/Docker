#!/usr/bin/env groovy
def label = "docker-jenkins-${UUID.randomUUID().toString()}"
// def registry = "km20/devops"
// def tag = "$registry:latest"
// def credentialsId = "docker_hub"

def app1_name = 'devops'
def app1_image_tag = "km20/${app1_name}:v${BUILD_NUMBER}"
def app1_dockerfile_name = 'Dockerfile'
def app1_container_name = 'devops'

podTemplate(label: label, 
    containers: [
        containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:alpine'),
        containerTemplate(name: 'kubectl', image: 'gcr.io/cloud-builders/kubectl', ttyEnabled: 'true', command: 'cat'),
        containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: 'true', command: 'cat'),
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', command: 'cat', ttyEnabled: 'true'),
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: 'true',
        envVars: [containerEnvVar(key: 'DOCKER_CONFIG', value: '/tmp/')]),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ],
  ) {
    node(label) 
    { 
        stage('Checkout-SCM') {
            container('git') {
            git credentialsId: 'git-creds', url: 'https://github.com/DevOps-MN/Docker.git'
            }
        }
        stage('Build Docker image') {
            container('docker') {
                script {
                    sh "docker build . -f ${app1_dockerfile_name} -t ${app1_image_tag}"
                }
            }
        }
        stage('Push Docker image') {
            container('docker') {
                echo "Push docker image..."
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'docker_hub'){
                        sh "docker push ${app1_image_tag}"   
                        }
                    }
            }
        }
        stage ('k8s-Deployment') {
            container('kubectl') {
            withKubeConfig([credentialsId: 'file-aws-config',
                caCertificate: env.k8s_caCertificate,
                serverUrl:   env.K8s_SERVER_URL,
                contextName: env.K8s_CONTEXT_NAME,
                clusterName: env.K8s_CLUSTER_NAME,
                ]) {
                    sh 'kubectl get pods'
                    sh "kubectl apply -f ${app1_name}.yaml"
                    sh "kubectl set image deployment/${app1_name} ${app1_container_name}=${app1_image_tag}"
                     }
            }
        }
        
        stage('Remove Unused docker image') {
            container('docker'){
                script{
                    sh "docker rmi ${app1_image_tag}"
                }
            }
        }
    }
}