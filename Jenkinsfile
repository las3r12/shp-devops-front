pipeline{
    agent none
    environment {
        PROJECT_DIR = '/var/www/ponomarenko_random_numbers/'
    }
    stages{
        stage("deps") {
            agent {
                docker{
                    image 'node:latest'
                    args '-u root'
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage("build") {
            agent {
                docker{
                    image 'node:latest'
                    args '-u root'
                }
            }
            steps {
                sh 'npm run build_prod'
            }
        }

        stage("sonar qube"){
            agent any
            steps {
                withCredentials([
                    string(credentialsId: "sonarqube_url", variable: "SONARQUBE_URL"),
                    usernamePassword(credentialsId: "sonar_numbers_front", usernameVariable: "PROJECT_KEY", passwordVariable: "PROJECT_TOKEN")
                    ]){
                    sh '''docker run \
                        --rm \
                        -e SONAR_HOST_URL="${SONARQUBE_URL}" \
                        -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=${PROJECT_KEY} -Dsonar.python.coverage.reportPaths=coverage.xml" \
                        -e SONAR_TOKEN="${PROJECT_TOKEN}" \
                        -v "${WORKSPACE}:/usr/src" \
                        sonarsource/sonar-scanner-cli
                    '''
                    }
            }
        }
        stage("deploy") {
            agent any
            steps {
                withCredentials(
                    [
                        string(credentialsId: "production_ip", variable: 'SERVER_IP'),
                        sshUserPrivateKey(credentialsId: "production_key", keyFileVariable: 'SERVER_KEY', usernameVariable: 'SERVER_USERNAME')
                    ]
                ) {
                    sh 'ls'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} mkdir -p ${PROJECT_DIR}'
                    sh 'scp -i ${SERVER_KEY} -r dist/* ${SERVER_USERNAME}@${SERVER_IP}:${PROJECT_DIR}'
                }
            }
        }
        stage("proxy config") {
            agent any
            steps {
                withCredentials(
                    [
                        string(credentialsId: "production_ip", variable: 'SERVER_IP'),
                        sshUserPrivateKey(credentialsId: "production_key", keyFileVariable: 'SERVER_KEY', usernameVariable: 'SERVER_USERNAME')
                    ]
                ) {
                    sh 'scp -i ${SERVER_KEY} ponomarenko-numbers.prod.mshp-devops.com.conf ${SERVER_USERNAME}@${SERVER_IP}:nginx'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} sudo certbot --nginx --non-interactive -d ponomarenko-numbers.prod.mshp-devops.com'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} sudo systemctl reload nginx'
                }
            }
        }
    }
}