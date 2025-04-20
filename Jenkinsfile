def repositoryName
pipeline {
    agent any
    environment {
        baseUrl = "https://github.com/Kriistoffer"
        allRepositories = "jenkins-demo,jenkins-demo-2,jenkins-demo-3"
        PACKAGEMANAGER = "npm"
        slackChannel = "team1-dependency_check"
    }
    stages {
        stage("Setup") {
            steps {
                script {
                    if (!fileExists("logs")) {
                        sh "mkdir logs"
                    }
                    env.allRepositories.tokenize(",").each { repo -> 
                        sh "git clone ${env.baseUrl}/${repo}"
                    }
                    // def skipper = env.skipperList.tokenize(",")
                }
            }
        }
        stage("Node") {
            steps {
                script {
                    def files = findFiles(glob: "**/package-lock.json")
                    echo "files: ${files}"

                    for (file in files) {
                        def parentDirectory = "${file.path}" - "/${file.name}"
                        dir("${parentDirectory}") {
                            sh "${env.PACKAGEMANAGER} install"
                            sh "${env.PACKAGEMANAGER} audit > ${WORKSPACE}/logs/audit_output.json"
                            sh "${env.PACKAGEMANAGER} outdated > ${WORKSPACE}/logs/outdated_output.json"

                            if (${env.PACKAGEMANAGER} == "npm") {
                                //NPM STEPS
                                //SLACKSEND
                                def node_vuln = readJSON(file: "${WORKSPACE}/logs/audit_output.json")
                                def node_outd = readJSON(file: "${WORKSPACE}/logs/outdated_output.json")

                                slackSend(color: "good",
                                channel: ${env.slackChannel},
                                message: "Vulnerabilities found for ${parentDirectory}: ${node_vuln.vulnerabilities.metadata.total}")

                                slackSend(color: "good",
                                channel: ${env.slackChannel},
                                message: "Outdated found for ${parentDirectory}: ${node_outd.size()}")
                            } else {
                                //YARN STEPS
                                //SLACKSEND
                                echo "This is a placeholder for the Yarn steps"
                            }
                        }
                    }
                }
            }
        }
        // stage("Dotnet") {
        //     steps {
        //         script {
        //             echo "This is a placeholder for the Dotnet stage"
        //         }
        //     }
        // }
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

          // def files = findFiles(glob: "**/testing.json")

                    // echo "Test: ${files}"
                    // try {
                    //     echo "Test first element: ${files[0]}"
                    // } catch (Exception ex) {
                    //     echo "No files could be found."
                    //     echo "Exception: ${ex}"
                    // }
            
                    // for (file in files) {
                    //     echo "Files: ${file.path}"
                    //     def test = (file.path =~ /^([^\/]+)/)[0][1]
                    //     echo "Corrected: ${test}"
                    // }