def regex = /(?<=\/(?!.*\/))(.*)(?=\.)/

pipeline {
    agent any
    tools {
        nodejs "nodejs"
        dotnetsdk "dotnet"
    }
    environment {
        node_repositories = '''https://github.com/Kriistoffer/jenkins-demo.git,
        https://github.com/Kriistoffer/jenkins-demo-2.git'''
    }
    stages {
        stage("Förberedelser") {
            steps {
                script {
                    if (!fileExists("logs")) {
                        sh "mkdir -p logs"
                        echo "Creating logs directory."
                    }
                }
            }
        }
        stage("Node: klona och scanna repositories") {
            steps {
                script {
                    env.node_repositories.tokenize(",").each { repo -> 
                        def repositoryName = (repo =~ /(?<=\/(?!.*\/))(.*)(?=\.)/)[0][1]

                        if (fileExists("${repositoryName}")) {
                            sh "git pull origin master"
                        } else {
                            sh "git clone ${repo}"
                        }

                        dir("${repositoryName}") {
                            sh "npm ci"
                            sh "npm audit --json > ${WORKSPACE}/logs/${repositoryName}_audit.json || true"
                        }
                    }
                }
            }
        }
        stage("Node: sammanställ sårbarheter") {
            steps {
                script {
                    def vuln = []
                    
                    env.node_repositories.tokenize(",").each { project -> 
                        def repositoryName = (project =~ regex)[0][1]
                        def result = readJSON(file: "${WORKSPACE}/logs/${repositoryName}_audit.json")
                        vuln.add("${repositoryName}: ${result.metadata.vulnerabilities.total} found") 
                    }

                    echo "vuln: ${vuln}"
                    def finalPrint = vuln.tokenize(",")
                    slackSend(channel: "#team2-dependency_check", color: "good", message: "${finalPrint}")
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
            // cleanWs()
            echo "Finished running."
        }
    }
}