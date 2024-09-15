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
        stage("Clone all repositories") {
            steps {
                script {
                    echo "Cloning repositories..."

                    node_repositories.tokenize(",").each { repository ->
                        echo "${repository.subString(${repository}.lastIndexOf("/") + 1, ${repository}.length())}"
                    }

                    // env.node_repositories.tokenize(",").each { repository ->
                    //     echo "Cloning ${repository} now..."
                    //     sh "mkdir -p ${repository}"
                    //     dir("${repository}") {
                    //         sh "git clone ${repository}"
                    //     }
                    //     echo "Finished cloning ${repository}"
                    // }
                }
            }
        }
        stage("Node: install node based repositories") {
            steps {
                script {
                    echo "Installing..."
                    env.node_repositories.tokenize(",").each { repository ->
                        echo "Installing ${repository} now..."
                        dir("${repository}") {
                            sh "npm install"
                        }
                        echo "Finished installing ${repository}."
                    }
                }
            }
        }
        stage("Node: version and audit check") {
            steps {
                echo "Auditing and checking dependency versions..."
                sh "mkdir -p logs/${BUILD_NUMBER}"
                script {
                    env.node_repositories.tokenize(",").each { repository ->
                        echo "Checking ${repository}..."

                        dir("${repository}") {
                            sh "npm outdated --json > ../logs/${BUILD_NUMBER}/${repository}_outdated_dependencies.json || true"
                            def outdated_output = readJSON(file: "../logs/${BUILD_NUMBER}/${repository}_outdated_dependencies.json")

                            //Format testing
                            // sh "npm outdated > ../logs/${BUILD_NUMBER}/${repository}_outdated_dependencies.txt || true"
                            // sh "npm audit > ../logs/${BUILD_NUMBER}/${repository}_vulnerabilities.txt || true"
                            
                            sh "npm audit --json > ../logs/${BUILD_NUMBER}/${repository}_vulnerabilities.json || true"
                            def audit_output = readJSON(file: "../logs/${BUILD_NUMBER}/${repository}_vulnerabilities.json")

                            slackSend(channel: "#team1-dependency_check", message: "- ${repository} - Outdated dependencies: ${outdated_output.size()}. Vulnerabilities found: ${audit_output.metadata.vulnerabilities.total}")
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
            // cleanWs()
            echo "Finished running."
        }
    }

}