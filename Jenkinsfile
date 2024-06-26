pipeline {
    agent any

    parameters {
        string(name: 'DATASET_NAME', description: 'Dataset name')
        string(name: 'TICKET_ID', description: 'Ticket ID')
    }

    environment {
        // DATAHUB_TOKEN = credentials('datahub-token')
        ZAMMAD_USR = credentials('zammad-usr')
        ZAMMAD_PW = credentials('zammad-pw')
	DOCKER_IMAGE = "${env.DOCKER_REGISTRY}/${env.GOLDEN_PROJECT_NAME}/denodo:latest"  
    }

    stages {
        stage('Pre-flight Check') {
            steps {
                script {
                    if (!params.DATASET_NAME) {
                        error "The DATASET_NAME parameter is missing. Please provide the dataset name."
                    }
                    if (!params.TICKET_ID) {
                        error "The TICKET_ID parameter is missing. Please provide the ticket ID."
                    }
                }
            }
        }

        stage('Run Dataset deploy') {
            steps {
                script {					
		    def dnsIP = sh(script: "kubectl get service -n kube-system kube-dns -o jsonpath='{.spec.clusterIP}'", returnStdout: true).trim()	
		    def now = new Date()
        	    def ts = now.format("yyMMddHHmm", TimeZone.getTimeZone('UTC'))


		    env.CONTAINER_NAME = "denodo_${env.BUILD_ID}"

                    // Run the container with necessary volumes and DNS settings, and execute the commands
                    sh """
                        docker run --name $CONTAINER_NAME -d -v /tmp:/tmp --dns=${dnsIP} $DOCKER_IMAGE tail -f /dev/null
                        docker exec $CONTAINER_NAME sh -c "mkdir -p /tmp/${env.ts}"
                        docker exec $CONTAINER_NAME sh -c "/opt/denodo/bin/export.sh --server //${env.DENODO_META_SANDBOX_URL}/admin --login admin --password admin --singleuser --repository-element admin:view:/${env.DATASET_NAME} --repository /tmp/${env.ts}"
                        docker exec $CONTAINER_NAME sh -c "/opt/denodo/bin/import.sh --server //${env.DENODO_META_PROD_URL}/admin?admin@admin --singleuser --repository /tmp/${env.ts} --element /databases/admin/views/${env.DATASET_NAME}.vql"
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    """
		    
                    // Run the Python script within the Docker container
        //             docker.withRegistry("https://${env.DOCKER_REGISTRY}") {
        //                 // Create a Docker image object
        //                 def denodoImage = docker.image("${env.DOCKER_REGISTRY}/${env.GOLDEN_PROJECT_NAME}/denodo:latest")
        //                 // Run the container with the script mounted and execute the Python script
        //                 denodoImage.inside("-v /tmp:/tmp --dns=${dnsIP}") {
        //                     sh """
			     //                                    mkdir -p /tmp/${env.ts}
								// /opt/denodo/bin/export.sh --server //${env.DENODO_META_SANDBOX_URL}/admin --login admin --password admin --singleuser --repository-element admin:view:/${env.DATASET_NAME} --repository /tmp/${env.ts}
								// /opt/denodo/bin/import.sh --server //${env.DENODO_META_PROD_URL}/admin?admin@admin --singleuser --repository /tmp/${env.ts} --element /databases/admin/views/${env.DATASET_NAME}.vql
							 //   """
        //                 }
        //             }
                }
            }
        }
		
		stage('Deploy data catalog dataset') {
            steps {
                script {					
					
                    // Run the Python script within the Docker container
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_REGISTRY_CREDENTIALS_ID) {
                        // Create a Docker image object
                        def pythonImage = docker.image("${env.DOCKER_REGISTRY}/${env.GOLDEN_PROJECT_NAME}/${env.GOLDEN_DOCKER_IMAGE}:${env.GOLDEN_DOCKER_TAG}")
                        // Run the container with the script mounted and execute the Python script
                        pythonImage.inside("-v ${env.WORKSPACE}:/app/workspace") {
                            sh """
                                python3 /app/workspace/data-catalog-deploy.py ticket_id=${env.TICKET_ID} token=${env.DATAHUB_TOKEN} zammad_usr=${env.ZAMMAD_USR} zammad_pw=${env.ZAMMAD_PW} zammad_url=${env.ZAMMAD_URL} dataset=${env.DATASET_NAME} datahub_url=${env.DATAHUB_URL}
		 					   """
                        }
                    }
                }
            }
        }
    }

   // post {
   //     always {
   //         // Optional: clean up
   //         // cleanWs()
   //     }
   // }
}
