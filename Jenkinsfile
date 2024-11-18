pipeline {
    agent any
    tools {
        nodejs "nodejs"
        dotnetsdk "dotnet"
    }
    environment {
        node_repositories = "https://github.com/Kriistoffer/jenkins-demo.git,https://github.com/Kriistoffer/jenkins-demo-2.git"
        node_repoNames = "jenkins-demo,jenkins-demo-2"
        dotnet_projects = ""
    }
    stages {
        stage("Clone all repositories") {
            steps {
                script {
                    env.node_repositories.tokenize(",").each { repository -> 
                        git clone ${repository}
                        def repositoryName = repository =~ /((?<=\/(?!.*\/))(.*)(?=\.))/

                        echo ${repositoryName}
                    }
                }
            }
        }
        stage("Node: install node based repositories") {
            steps {
                script {
                    echo "Adding stage later, if needed."
                }
            }
        }
        stage("Node: version and audit check") {
            steps {
                script {
                    echo "Adding stage later, if needed."
                }
            }
        }
    }
    post {
        success {
            script {
                def now = new Date()
                slackSend(channel: "#team2-dependency_check", color: "good", message: "${JOB_NAME} has finished running at ${now.format("yyyy-MM-dd HH:mm", TimeZone.getTimeZone("GMT+2"))}. Logs available at ${BUILD_URL}execution/node/3/ws/logs/${BUILD_NUMBER}")
            }
        }
        failure {
            script {
                def now = new Date()
                slackSend(channel: "#team2-dependency_check", color: "bad", message: "Something has caused a failure when running ${JOB_NAME} at ${now.format("yyyy-MM-dd HH:mm", TimeZone.getTimeZone("GMT+2"))}. The failed build can be found here: ${BUILD_URL}console")
            }
        }
        always {
            cleanWs()
            echo "Finished running."
        }
    }
}