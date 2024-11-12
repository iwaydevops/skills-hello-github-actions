pipeline {
    agent any
    
    parameters {
        // Defining parameters that can be used by the user to input values
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/your-repository.git', description: 'Git Repository URL')
        string(name: 'START_DATE', defaultValue: '', description: 'Start Date (Format: YYYY-MM-DD)')
        string(name: 'END_DATE', defaultValue: '', description: 'End Date (Format: YYYY-MM-DD)')
        string(name: 'EMAIL', defaultValue: '', description: 'Email Address to send the Git log report')
    }

    environment {
        // Optionally define any environment variables you may need
        GIT_REPO_DIR = 'repo'
    }

    stages {
        stage('Clone Git Repository') {
            steps {
                script {
                    // Clone the Git repository
                    echo "Cloning Git repository from ${params.GIT_REPO_URL}"
                    git url: params.GIT_REPO_URL, branch: 'main', changelog: false
                }
            }
        }

        stage('Generate Git Log') {
            steps {
                script {
                    // Validate the date range input
                    if (!params.START_DATE?.trim() || !params.END_DATE?.trim()) {
                        error "Please provide both start and end dates in the format YYYY-MM-DD."
                    }

                    // Run Git log command for the specified date range
                    echo "Generating Git log from ${params.START_DATE} to ${params.END_DATE}"
                    def gitLogCommand = """git log --since="${params.START_DATE}" --until="${params.END_DATE}" --oneline"""
                    def gitLog = sh(script: gitLogCommand, returnStdout: true).trim()

                    if (gitLog) {
                        echo "Git log generated."
                        currentBuild.description = "Git log from ${params.START_DATE} to ${params.END_DATE}."
                    } else {
                        echo "No commits found in the given date range."
                        currentBuild.description = "No commits in the specified date range."
                    }

                    // Save the Git log to a file
                    writeFile file: 'git_log.txt', text: gitLog
                }
            }
        }

        stage('Send Email') {
            steps {
                script {
                    // Check if the user provided an email address
                    if (params.EMAIL?.trim()) {
                        // Send an email with the Git log attached
                        emailext(
                            subject: "Git Log from ${params.START_DATE} to ${params.END_DATE}",
                            body: "Please find the attached Git log for the specified date range.",
                            to: params.EMAIL,
                            attachmentsPattern: 'git_log.txt'
                        )
                    } else {
                        echo "No email address provided. Skipping email notification."
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up any generated files or perform any necessary steps
            deleteDir() // Clean up the workspace
        }

        success {
            echo "Pipeline executed successfully."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
