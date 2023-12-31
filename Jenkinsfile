pipeline{
    agent any

            parameters{
                choice(name: 'action', choices: 'create\ndelete', description:'choose create/delete')
            }
            environment{
                registryname = "myacrregistryjuly27"
                registrycredentials = 'myacrregistryjuly27'
                registryurl = 'myacrregistryjuly27.azurecr.io'
                dockerImage=""
            }

  stages {
    stage("checkouts"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                git branch: 'main', url: 'https://github.com/kavin-bot/Java_app2023.git'
            }
        }
    } 
    /*stage("Maven: UnitTest"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                sh 'mvn test'
            }
        }
    }
    stage("IntegrationTest:Maven"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                sh 'mvn verify -DskipUnitTests'
            }
        }
    } */
    stage("StaticCodeAnalysis"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonar-api'){
                     sh 'mvn clean package sonar:sonar'
                }
                
            }
        }
    } 
    stage("QualityGate"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
            }
        }
    } 
    stage("mavenBuild"){
        when{ expression { params.action =='create'}}
        steps{
            script{
               sh 'mvn clean install'
            }
        }
    } 
   /* stage("DockerBuild"){
        when{ expression { params.action =='create'}}
        steps{
            script{
               sh """

               docker build -t dockerkavin/javaapp:${env.BUILD_ID} .
               """
            }
        }
    } */
    stage("ACR_DockerBuild"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                dockerImage= docker.build registryname
            }
        }

    }
/*  stage("DockerImagescanTrivy"){
        when{ expression { params.action =='create'}}
        steps{
            script{  
               sh "
            {
              trivy image dockerkavin/javaapp:${env.BUILD_ID} >scan.txt
               cat scan.txt
            }"
           
            }
        }
    } */
   /* stage("DockerImagescanTrivy"){
        when{ expression { params.action =='create'}}
        steps{
            script{  
               sh """
            {
              trivy image $dockerImage >scan.txt
               cat scan.txt
            }"""
           
            }
        }
    } */
    /*stage("DockerPush"){
        when{ expression { params.action =='create'}}
        steps{
            script{
               withCredentials([usernamePassword(credentialsId: 'dockerkavin', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                sh """ 
                docker login -u '$USER' -p '$PASS'
                docker push dockerkavin/javaapp:${env.BUILD_ID}
                """
                    } 
            }
        }
    } */

    //pushing the image to ACR
    stage("ACR_Dockerpush"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                docker.withRegistry("http://${registryurl}",registrycredentials){
                    dockerImage.push()
                }
            }
        }

    } 
    stage("Deploy in AKS"){
        when{ expression { params.action =='create'}}
        steps{
            script{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                  sh'kubectl apply -f deployment.yaml'
               }
            }
        }
    }
 }
}

