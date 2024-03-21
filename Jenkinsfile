pipeline {
    agent any

    parameters {
        string(name: 'TICKET_ID', description: 'Ticket ID')
    }

    environment {
        SCRIPT_PATH = 'src/dremio_cloner.py' // Path to your Python script in the Git repository
        DREMIO_USR_SANDBOX = credentials('dremio-usr-sandbox')
        DREMIO_PW_SANDBOX = credentials('dremio-pw-sandbox')
		DREMIO_USR_PROD = credentials('dremio-usr-prod')
        DREMIO_PW_PROD = credentials('dremio-pw-prod')
        DATAHUB_TOKEN = credentials('datahub-token')
    }

    stages {
        stage('Pre-flight Check') {
            steps {
                script {
                    if (!params.TICKET_ID) {
                        error "The TICKET_ID parameter is missing. Please provide the ticket ID."
                    }
                }
            }
        }

        stage('Run Dataset deploy') {
            steps {
                script {					
					
                    // Run the Python script within the Docker container
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_REGISTRY_CREDENTIALS_ID) {
                        // Create a Docker image object
                        def denodoImage = docker.image("${env.DOCKER_REGISTRY}/${env.GOLDEN_PROJECT_NAME}/denodo:latest")
                        // Run the container with the script mounted and execute the Python script
                        denodoImage.inside("-v ${env.WORKSPACE}:/tmp") {
                            sh """
								/opt/denodo/bin/export.sh --server //datamgmtdemo01.eastasia.cloudapp.azure.com:30999/admin --login admin --password admin --singleuser --repository-element admin:view:/data_catalog_draft --repository /tmp
							   """
                        }
                    }
                }
            }
        }
		
//		stage('Deploy data catalog dataset') {
//            steps {
//                script {					
//					
//                    // Run the Python script within the Docker container
//                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_REGISTRY_CREDENTIALS_ID) {
//                        // Create a Docker image object
//                        def pythonImage = docker.image("${env.DOCKER_REGISTRY}/${env.GOLDEN_PROJECT_NAME}/${env.GOLDEN_DOCKER_IMAGE}:${env.GOLDEN_DOCKER_TAG}")
//                        // Run the container with the script mounted and execute the Python script
//                        pythonImage.inside("-v ${env.WORKSPACE}:/app/workspace") {
//                            sh """
//                                python3 /app/workspace/data-catalog-deploy.py batch_key=${env.BATCH_KEY} token=${env.DATAHUB_TOKEN}
//							   """
//                        }
//                    }
//                }
//            }
//        }
    }

    post {
        always {
            // Optional: clean up
           // cleanWs()
        }
    }
}