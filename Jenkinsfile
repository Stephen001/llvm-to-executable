pipeline {
    agent any
    
    stages {
        stage('Create Kubernetes Namespace') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.newProject(env.BUILD_TAG)
                        openshift.withProject(env.BUILD_TAG) {
                            def data = readYaml file: 'image-stream.yaml'
                            openshift.apply(data)
                            data = readYaml file: 'build-config.yaml'
                            openshift.apply(data)
                        }
                    }
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(env.BUILD_TAG) {
                            def imageBuild = openshift.selector('bc/llvm-to-executable')
                            def buildExecution = imageBuild.startBuild()
                            buildExecution.logs('-f')
                            openshift.failUnless(buildExecution.status == 0)
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                openshift.withCluster() {
                    openshift.delete('project', env.BUILD_TAG)
                }
            }
        }
    }
}
