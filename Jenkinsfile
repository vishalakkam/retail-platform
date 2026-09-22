pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOYMENT_ACTION',
            choices: ['DEPLOY', 'ROLLBACK'],
            description: 'Choose deployment action'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['UAT', 'PRODUCTION'],
            description: 'Choose deployment environment'
        )

        string(
            name: 'VERSION',
            defaultValue: '4.2.1',
            description: 'Application version'
        )

        choice(
            name: 'CONFIRM_PROD',
            choices: ['NO', 'YES'],
            description: 'Production deployment confirmation'
        )
    }

    stages {

        stage('Validate Parameters') {
            steps {
                script {
                    echo "Action: ${params.DEPLOYMENT_ACTION}"
                    echo "Environment: ${params.ENVIRONMENT}"
                    echo "Version: ${params.VERSION}"

                    if (params.ENVIRONMENT == 'PRODUCTION' &&
                        params.CONFIRM_PROD != 'YES') {
                        error('Production deployment requires CONFIRM_PROD=YES')
                    }

                    echo 'Parameter validation successful'
                }
            }
        }

        stage('Validate Version') {
            steps {
                script {
                    def tagName = "v${params.VERSION}"

                    if (bat(
                        script: "git rev-parse --verify refs/tags/${tagName}",
                        returnStatus: true
                    ) != 0) {
                        error("Git tag ${tagName} does not exist")
                    }

                    echo "Git tag ${tagName} exists"
                }
            }
        }

        stage('Checkout Version') {
            steps {
                bat 'git fetch --tags'
                bat 'git checkout tags/v%VERSION%'
                echo "Checked out version v${params.VERSION}"
            }
        }

        stage('Identify Git Commit') {
            steps {
                script {
                    def commitId = bat(
                        script: 'git rev-parse HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Deploying Git commit: ${commitId}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "retail-platform:${params.VERSION}-${env.BUILD_NUMBER}"

                    echo "Building Docker image: ${imageTag}"
                }

                bat '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" build -t retail-platform:%VERSION%-%BUILD_NUMBER% .'
            }
        }

        stage('Record Previous Image') {
            steps {
                script {
                    def previousImage = bat(
                        script: '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" inspect --format="{{.Config.Image}}" retail-platform-prod',
                        returnStdout: true
                    ).trim()

                    env.PREVIOUS_IMAGE = previousImage

                    echo "Previous production image: ${env.PREVIOUS_IMAGE}"
                }
            }
        }

        stage('Deploy New Container') {
            steps {
                script {
                    def imageTag = "retail-platform:${params.VERSION}-${env.BUILD_NUMBER}"

                    echo "Starting new container: ${imageTag}"

                    bat(
                        returnStatus: true,
                        script: '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" rm -f retail-platform-new'
                    )

                    bat "\"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe\" run -d --name retail-platform-new -p 8083:80 ${imageTag}"

                    echo 'New container started on port 8083'
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    try {
                        echo 'Checking new application health...'

                        if (params.VERSION == '4.2.2') {
                            echo 'FAILURE INJECTION: forcing health check failure for 4.2.2'
                            bat 'curl --fail --silent http://localhost:8083/nonexistent-health-check'
                        } else {
                            bat 'curl --fail --silent http://localhost:8083'
                        }

                        echo 'Health check successful'
                    }
                    catch (err) {
                        echo 'Health check FAILED'
                        echo 'Starting automatic rollback...'

                        bat(
                            returnStatus: true,
                            script: '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" rm -f retail-platform-new'
                        )

                        echo "Previous production image remains active: ${env.PREVIOUS_IMAGE}"

                        echo 'Checking previous production version...'

                        bat 'curl --fail --silent http://localhost:8082'

                        echo 'Rollback verified successfully.'

                        error('Deployment failed. Automatic rollback completed.')
                    }
                }
            }
        }

        stage('Replace Production') {
            steps {
                script {
                    def imageTag = "retail-platform:${params.VERSION}-${env.BUILD_NUMBER}"

                    echo 'New image passed health check.'
                    echo 'Replacing old production container...'

                    bat '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" stop retail-platform-prod'

                    bat '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" rm retail-platform-prod'

                    bat "\"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe\" run -d --name retail-platform-prod -p 8082:80 ${imageTag}"

                    bat(
                        returnStatus: true,
                        script: '"C:\\Users\\Vishal Akkam\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" rm -f retail-platform-new'
                    )

                    echo 'New version is now running in production.'
                }
            }
        }
    }
}