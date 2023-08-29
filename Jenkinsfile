pipeline {
   environment {
       workspaceDir = "/home/jenkins_slave/test-workspace/testboot"                               //can be used in whole pipeline
   }
   triggers {
        pollSCM('H/2 * * * *') // Poll SCM every 2 minutes
   }

    agent {
        node {
            label 'cts' // Replace with the label of your Jenkins slave
            customWorkspace '${workspaceDir}'
        }
    }

    stages {
       stage('Check Workspace Directory') {
            steps {
                script {
                    if (fileExists(workspaceDir)) {
                        echo "Workspace directory exists: ${workspaceDir}"
                        env.WORKSPACE_EXISTS = 'true'
                    } else {
                        echo "Workspace directory does not exist: ${workspaceDir}"
                        env.WORKSPACE_EXISTS = 'false'
                    }
                }
            }
        }
        stage('Check for Stats Folder') {
            when {
                expression { return env.WORKSPACE_EXISTS == 'true' }
            }
            steps {
                script {
                    def statsFolder = "${workspaceDir}/test_git1"
                    if (fileExists(statsFolder)) {
                        echo "Stats folder exists. Proceeding with the build..."
                        env.STATS_FOLDER_EXISTS = 'true'
                    } else {
                        echo "Stats folder does not exist. Skipping build."
                        env.STATS_FOLDER_EXISTS = 'false'
                        currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }
        stage('Checkout Repository and Run Commands') {
            steps {
                script {
                    
                    dir(workspaceDir) {
                        // Clone the Git repository into the workspace
                        checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/novablack6/gradle-simple.git']], extensions: [[$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [[path: 'git_test/*']]]]])
                        sh 'echo "Current working directory: $(pwd)"'
                    }
                }
            }
        }
        stage('Check for Changes') {
            steps {
                script {
                    def changedFiles = []
                    dir(workspaceDir) {
                        // Get a list of changed files in the "data" folder
                        changedFiles = sh(script: "git diff --name-only origin/master...HEAD test_git/", returnStdout: true).trim().split('\n')
                    }
                    
                    if (changedFiles.size() > 0) {
                        echo "Changes detected in the 'test_git' folder. Triggering build..."
                        env.CHANGES_DETECTED = 'true'
                    } else {
                        echo "No changes in the 'test_git' folder. Skipping build."
                        env.CHANGES_DETECTED = 'false'
                        currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }
        
        stage('Build') {
            when {
                expression { return env.CHANGES_DETECTED == 'true' && env.STATS_FOLDER_EXISTS == 'true' }
            }
            steps {
                script {
                    dir(workspaceDir) {
                        // Run gradle clean build
                        sh 'gradle clean build'
                    }
                }
            }
        }
        
        stage('Run Workspace Commands') {
            when {
                expression { return env.CHANGES_DETECTED == 'true' && env.STATS_FOLDER_EXISTS == 'true' }
            }
            steps {
                script {
                    dir(workspaceDir) {
                        echo 'Hello, work!'
                        sh 'echo "Current working directory: $(pwd)"'
                    }
                }
            }
        }
    }
}
