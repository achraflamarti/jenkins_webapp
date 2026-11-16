pipeline {
    agent any

	tools {
        maven 'Maven310' // doit correspondre au nom donné dans Global Tool Configuration
        jdk 'Java17'
    }
    environment {
        APP_PORT = '8383'
    }

    stages {

        stage('1. Checkout') {
            steps {
                git branch: 'master', // ou 'master', adaptez à votre branche
                    url: 'https://github.com/MezghichGit/jenkins_webapp.git'
                    
            }
        }

        stage('2. Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('3. Run JAR') {
            steps {
                script {
                    // Récupère dynamiquement le nom du jar généré dans target/
                    def jarFile = sh(script: "ls target/*.jar | grep -v original | head -1", returnStdout: true).trim()

                    // Stoppe une éventuelle instance précédente sur le même port
                    sh """
                        PID=\$(lsof -t -i:${APP_PORT} || true)
                        if [ ! -z "\$PID" ]; then
                            echo "Arrêt de l'ancien processus sur le port ${APP_PORT} (PID \$PID)"
                            kill -9 \$PID || true
                        fi
                    """

                    // Lance le jar en arrière-plan, écoute sur toutes les interfaces
                    sh """
                        nohup java -jar ${jarFile} --server.port=${APP_PORT} --server.address=0.0.0.0 > app.log 2>&1 &
                        sleep 5
                        echo "Application démarrée sur le port ${APP_PORT}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Déploiement réussi. Accessible sur http://jenkins.sip-academy.com:${APP_PORT}"
        }
        failure {
            echo "Le build ou le déploiement a échoué."
        }
    }
}