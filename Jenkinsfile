pipeline {
    agent {
        label "maven"
    }
    options { 
        skipDefaultCheckout()
        disableConcurrentBuilds()
    }
    stages {
        stage("Checkout") {
            steps {
                checkout(scm)   
            }
        }

        stage('Compile') {
            steps {
                sh "mvn package -DskipTests"
            }
       }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
       stage('Build Image') {
            steps {
                script {
                    openshift.withCluster {
                        openshift.withProject {
                            env.TAG= readMavenPom().getVersion()
                            openshift.selector("bc","hello-openshift").startBuild("--from-dir=./target","--wait=true")
                        }

                    }
                }
            }
           }
    }
}