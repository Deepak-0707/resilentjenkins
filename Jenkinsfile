pipeline{
    agent any
    parameters{
        choice(name:'ENV',choices:['dev','prod'],description:"Environement of deployment")
        booleanParam(name:"SIMULATE_FAILURE",defaultValue:false)
    }
    environment{
        DOCKER='"C:/Program Files/Docker/Docker/resources/bin/docker.exe"'
        IMAGE_TAG="app:${env.BUILD_NUMBER}"
    }
    options{
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }
    stages{
        stage("Cleaning Workspace and Pulling Code"){
            steps{
                cleanWs()
                checkout scm
            }
        }
        stage("building Image"){
            steps{
                bat "%DOCKER% build -t app ."
            }
        }
        stage("Simualated Test Phase"){
            steps{
                script{
                    if(params.SIMULATE_FAILURE){
                        error("Simulated Test Failure.")
                    }
                    else{
                        echo "Test Passed"
                    }
                }
            }
        }
        stage("Push Image with Retry"){
            steps{
                retry(2){
                    bat "%DOCKER% tag %IMAGE_TAG% deepakm06/app:latest"
                }
            }
        }
        stage("Prod Approval"){
            when{
                expression {params.ENV=="prod"}
            }
            steps{
                input message:"Approve production deployment?"
            }
        }
        stage("Deploy"){
            steps{
                timeout(time:10,unit:"SECONDS"){
                    echo "Deploying to ${params.ENV}"
                    sleep 3
                }
            }
        }
        stage("Health Check"){
            steps{
                script{
                    if(params.SIMULATE_FAILURE){
                        error("Health check failed")
                    }
                    else{
                        echo "Application healthy"
                    }
                }
            }
        }
        stage("ROllback"){
            when{
                expression {currentBuild.result=='FAILURE'}
            }
            steps{
                echo "Rolling back"
            }
        }
    }
    post{
        always{
            cleanWs()

        }
        success{
            echo "Application completed all its task"
        }
        failure{
            echo "Application didint finish its task"
        }
    }
}