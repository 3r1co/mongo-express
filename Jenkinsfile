#!/usr/bin/groovy
//def envStage = "${env.JOB_NAME}-staging"
def envStage = "re-risk-engine-staging"
def envProd = "${env.JOB_NAME}-production"

node ('kubernetes'){ 

  git 'https://github.com/jmlambert78/mongo-express'

  stage 'canary release'
    if (!fileExists ('Dockerfile')) {
      writeFile file: 'Dockerfile', text: 'FROM node:5.3-onbuild'
    }
    
    def newVersion = "1.0.${env.BUILD_NUMBER}"
    
    // Pushing image using docker client
	kubernetes.pod('dockerpushpod').withImage('docker:1.9.1')
	 .withPrivileged(true)
	 .withHostPathMount('/var/run/docker.sock','/var/run/docker.sock')
	 .withEnvVar('DOCKER_CONFIG','/home/jenkins/.docker/')
	 .withServiceAccount('jenkins')
	 .inside {
		 sh "docker build --no-cache --pull -t ${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}:${newVersion} ."
		 sh "docker push ${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}:${newVersion}"
	 }

    def rc = getKubernetesJson {
      port = 8080
      label = 'node'
      icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/nodejs.svg'
      version = newVersion
      imageName = clusterImageName
    }

  stage 'Rolling upgrade Staging'
    kubernetesApply(file: rc, environment: envStage)

 
}
