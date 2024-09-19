pipeline {
    agent any
    tools {
        nodejs "nodejs"
        dotnetsdk "dotnet"
    }
    environment {
        node_repositories = "https://github.com/Kriistoffer/jenkins-demo,https://github.com/Kriistoffer/jenkins-demo-2"
        dotnet_projects = ""
    }
    stages {
        stage("Clone all repositories") {
            steps {
                script {
                    env.node_repositories.tokenize(",").each { repository -> 
                        sh "git clone ${repository}"
                        echo "Finished cloning ${repository}"
                    }
                }
            }
        }
        stage("Node: install node based repositories") {
            steps {
                script {
                    echo "Installing..."
                    env.node_repositories.tokenize(",").each { repository ->
                        def repoName = repository.split('Kriistoffer/')
                        dir("${repoName[1]}") {
                            sh "npm install"
                            echo "Finished installing ${repoName[1]}."
                        }
                    }
                }
            }
        }
        stage("Node: version and audit check") {
            steps {
                echo "Auditing and checking dependency versions..."
                script {
                    sh "mkdir -p ${WORKSPACE}/logs/${BUILD_NUMBER}"
                    env.node_repositories.tokenize(",").each { repository ->
                        echo "Checking ${repository}..."
                        def repoName = repository.split('Kriistoffer/')

                        dir("${repoName[1]}") {
                            sh "npm outdated --json > ${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_outdated_dependencies.json || true"
                            def outdated_output = readJSON(file: "${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_outdated_dependencies.json")

                            //Format testing
                            // sh "npm outdated > ${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_outdated_dependencies.txt || true"
                            // sh "npm audit > ${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_vulnerabilities.txt || true"
                            
                            sh "npm audit --json > ${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_vulnerabilities.json || true"
                            def audit_output = readJSON(file: "${WORKSPACE}/logs/${BUILD_NUMBER}/${repoName[1]}_vulnerabilities.json")

                            slackSend(channel: "#team2-dependency_check", message: "- ${repoName[1]} - Outdated dependencies: ${outdated_output.size()}. Vulnerabilities found: ${audit_output.metadata.vulnerabilities.total}")
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                def now = new Date()
                slackSend(channel: "#team2-dependency_check", color: "good", message: "${JOB_NAME} has finished running at ${now.format("yyMMdd-HH:mm", TimeZone.getTimeZone("GMT+2"))}. Logs available at ${BUILD_URL}execution/node/3/ws/logs/${BUILD_NUMBER}")
            }
        }
        failure {
            script {
                def now = new Date()
                slackSend(channel: "#team2-dependency_check", color: "bad", message: "Something has caused a failure when running ${JOB_NAME} at ${now.format("yyMMdd-HH:mm", TimeZone.getTimeZone("GMT+2"))}. The failed build can be found here: ${BUILD_URL}console")
            }
        }
        always {
            cleanWs()
            echo "Finished running."
        }
    }
}