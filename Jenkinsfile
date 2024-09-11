pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['deploy', 'delete'],
            description: 'Choose whether to deploy or delete the namespace'
        )
    }

    environment {
        KUBECONFIG = credentials('hetzner-k8s')  // Utiliser kubeconfig depuis les credentials de Jenkins
        BRANCH_NAME = "${env.GIT_BRANCH}"  // Récupérer le nom de la branche depuis l'environnement Git
        NAMESPACE = "my-app-${BRANCH_NAME}"  // Namespace basé sur le nom de la branche
    }

    stages {
        stage('Determine Action') {
            steps {
                script {
                    if (params.ACTION == 'deploy') {
                        echo "Action: Deploy"
                    } else if (params.ACTION == 'delete') {
                        echo "Action: Delete"
                    } else {
                        error "Invalid action specified!"
                    }
                }
            }
        }

        stage('Create Namespace') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                script {
                    echo "Creating Kubernetes Namespace: ${NAMESPACE}"
                    // Créer le namespace
                    sh "kubectl create namespace ${NAMESPACE} || echo 'Namespace already exists, continuing...'"
                }
            }
        }

        stage('Apply Kubernetes Manifests') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                script {
                    echo "Applying Kubernetes Manifests in Namespace: ${NAMESPACE}"
                    // Appliquer les manifests dans le namespace créé
                    sh "kubectl apply -f . -n ${NAMESPACE}"
                }
            }
        }

        stage('Display Resources in Namespace') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                script {
                    echo "Displaying resources in Namespace: ${NAMESPACE}"
                    // Afficher les ressources du namespace
                    sh "kubectl get all -n ${NAMESPACE}"
                }
            }
        }

        stage('Delete Namespace') {
            when {
                expression { params.ACTION == 'delete' }
            }
            steps {
                script {
                    echo "Deleting Kubernetes Namespace: ${NAMESPACE}"
                    // Supprimer le namespace
                    sh "kubectl delete namespace ${NAMESPACE}"
                }
            }
        }
    }

    post {
        always {
            script {
                // En cas de déploiement, collecter les logs et événements
                if (params.ACTION == 'deploy') {
                    echo "Collecting Namespace Logs and Events for: ${NAMESPACE}"
                    sh "kubectl get events -n ${NAMESPACE} || echo 'No events found.'"
                    sh "kubectl logs -n ${NAMESPACE} --all-containers=true || echo 'No logs available.'"
                }
            }
        }

        success {
            echo "Pipeline executed successfully!"
        }

        failure {
            // Afficher les détails en cas d'échec
            echo "Pipeline failed! Gathering additional information."
            sh "kubectl describe all -n ${NAMESPACE} || echo 'Namespace does not exist or no resources found.'"
        }
    }
}
