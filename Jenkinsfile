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
    }
}