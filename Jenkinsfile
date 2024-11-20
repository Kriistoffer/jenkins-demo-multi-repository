def regex = /(?<=\/(?!.*\/))(.*)(?=\.)/

pipeline {
    agent any
    tools {
        nodejs "nodejs"
        dotnetsdk "dotnet"
    }
    environment {
        node_repositories = '''
        https://github.com/Kriistoffer/jenkins-demo.git,
        https://github.com/Kriistoffer/jenkins-demo-2.git,
        https://github.com/Kriistoffer/jenkins-demo-3.git'''
        node_subdirectories = '''myapp,/,mysecondapp/src'''
    }
    stages {
        stage("Förberedelser") {
            steps {
                script {
                    if (!fileExists("logs")) {
                        sh "mkdir -p logs"
                    }
                }
            }
        }
        stage("Node: klona och scanna repositories") {
            steps {
                script {
                    env.node_repositories.tokenize(",").each{ repo -> 
                        sh "git clone ${repo}"

                        def directory = (repo =~ regex)
                        // env.node_subdirectories.tokenize(",").each{ subdirectory -> 
                        //     dir("${directory}/${subdirectory}") {
                        //         sh "pwd"
                        //     }
                        // }
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                def now = new Date()
                echo "success"
            }
        }
        failure {
            script {
                def now = new Date()
                echo "failure"
                cleanWs()
                // slackSend(channel: "#team2-dependency_check", color: "bad", message: "Something has caused a failure when running ${JOB_NAME} at ${now.format("yyyy-MM-dd HH:mm", TimeZone.getTimeZone("GMT+2"))}. The failed build can be found here: ${BUILD_URL}console")
            }
        }
        always {
            // cleanWs()
            echo "Finished running."
        }
    }
}