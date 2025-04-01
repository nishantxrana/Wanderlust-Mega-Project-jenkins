@Library('EKS-Shard-Library') _
pipeline {
    agent {label 'worker'}
    
    environment{
        SONAR_HOME = tool "Sonar"
        FRONTEND_DOCKER_TAG = "v${env.BUILD_ID}"
        BACKEND_DOCKER_TAG = "v${env.BUILD_ID}"
    }
    
    // parameters {
    //     string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    //     string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    // }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/nishantxrana/Wanderlust-Mega-Project-jenkins.git","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","EKS_Jenkins","EKS_Jenkins")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            docker_build("eks-backend","${env.BACKEND_DOCKER_TAG}","infraflux")
                        }
                    
                        dir('frontend'){
                            docker_build("eks-frontend","${env.FRONTEND_DOCKER_TAG}","infraflux")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("eks-backend","${env.BACKEND_DOCKER_TAG}","infraflux","DockerHub") 
                    docker_push("eks-frontend","${env.FRONTEND_DOCKER_TAG}","infraflux", "DockerHub")
                }
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "EKS-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${env.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${env.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
