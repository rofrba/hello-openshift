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
                            openshift.tag("hello-openshift/hello-openshift:latest", "hello-openshift:${env.TAG}")
                        }

                    }
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    openshift.withCluster {
                        openshift.withProject("hello-openshift") {
                            openshift.set("triggers", "dc/hello-openshift", "--remove-all")
                            openshift.set("triggers", "dc/hello-openshift", "--from-image=hello-openshift:latest", "-c hello-openshift")
                            openshift.selector("dc", "hello-openshift").rollout().status()
                        }
                    }
                }
                
            }

        }

        stage('Promote Stage'){
            steps {
                input("Promote to Stage?") {
                    script {
                    openshift.withCluster {
                    openshift.withProject("hello-openshift") {
                        openshift.tag("hello-openshift:${ENV.TAG}", "hello-openshift-stage/hello-openshift:${ENV.TAG}")
                        }
                    }
                }
                

                }
            }
        }

        stage('Deploy STAGE'){
            steps{
                script{
                    openshift.withCluster {
                        openshift.withProject('hello-openshift-stage') {
                            openshift.set("triggers", "dc/hello-openshift", "--remove-all")
                            openshift.set("triggers", "dc/hello-openshift", "--from-image=hello-openshift:latest", "-c hello-openshift")
                            openshift.selector("dc", "hello-openshift").rollout().status()
                        }
                    }

                }
            }

        }
    }
}