def repositoryName
pipeline {
    agent any
    environment {
        node_repositories = "https://github.com/Kriistoffer/jenkins-demo.git,https://github.com/Kriistoffer/jenkins-demo-2.git,https://github.com/Kriistoffer/jenkins-demo-3.git"
    }
    stages {
        stage("Setup") {
            steps {
                script {
                    if (!fileExists("logs")) {
                        sh "mkdir logs"
                    }

                    env.node_repositories.tokenize(",").each { repo -> 
                        sh "git clone ${repo}"
                    }
                }
            }
        }
        stage("Testing") {
            steps {
                script {
                    def files = findFiles(glob: "**/package-lock.json")
                    // def regex = /[^\/]*/
                    def line = "this is a test line"
                    def test = (line =~ /[^\/]*/)[0][1]
                    echo "${test}"



                    // for (file in files) {
                    //     echo "Files: ${file.path}"
                    //     def test = (file.path =~ regex)[0][1]
                    //     echo "Corrected: ${test}"
                    // }
                }
            }
        }
    }
    post {
        success {
            script {
                echo "success"
            }
        }
        failure {
            script {
                echo "failure"
            }
        }
        always {
            cleanWs()
            echo "Finished running."
        }
    }
}