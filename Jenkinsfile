pipeline{
 environment {
        dockerUserName="alannewton"
        credentialsIdGCP = "exec-cohort-11"
        namespace = "lbg-12"
        // e.g. lbg-1 for learner1, lbg-2 for learner2
        projectId= "exec-cohort-11"
        
        imageName = "vatcalc"
        registry = "${dockerUserName}/${imageName}"
        registryCredentials = "dockerhub_id"
        clusterName = "lbg-gke"
        location = "europe-west1"
    }

    agent any
        stages {
           stage('Install Dependencies') {
                steps {
                // Install the ReactJS dependencies
                sh "npm install"
                }
            }
            stage('Run Tests') {
                steps {
                // Run the ReactJS tests
                sh "npm test"
                }
            }
            stage('SonarQube Analysis') {
                environment {
                    scannerHome = tool 'sonarqube'
                }
                steps {
                    withSonarQubeEnv('sonar-qube-1') {        
                    sh "${scannerHome}/bin/sonar-scanner"
                    }
                    timeout(time: 10, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                    }
                }
            }
         
            stage ('Build Docker Image'){
                steps{
                    script {
                        dockerImage = docker.build(registry)
                    }
                }
            }

            stage ("Push to Docker Hub"){
                steps {
                    script {
                        docker.withRegistry('', registryCredentials) {
                            dockerImage.push("${env.BUILD_NUMBER}")
                            dockerImage.push("latest")
                        }
                    }
                }
            }

            stage('Deploy to GKE') {
                steps{
                    sh "sed -i 's|dockerid/image:latest|${dockerUserName}/${imageName}:${env.BUILD_ID}|g' deployment.yaml"
                    step([$class: 'KubernetesEngineBuilder', 
                    projectId: projectId, 
                    clusterName: clusterName, 
                    location: location, 
                    namespace: namespace,
                    manifestPattern: 'deployment.yaml', 
                    credentialsId: credentialsIdGCP, 
                    verifyDeployments: true])
                }
            }

            stage ("Clean up"){
                steps {
                    script {
                        sh 'docker image prune --all --force --filter "until=48h"'
                           }
                }
            }
        }
}
