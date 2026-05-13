pipeline {
    agent {
        label 'linux'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['DEV', 'QA', 'PROD'],
            description: 'Ambiente donde se ejecutará Flyway'
        )
    }

    environment {
        FLYWAY = '/opt/devops/tools/flyway/flyway'
       PROJECT_DIR = "${WORKSPACE}"
        BACKUP_DIR = '/opt/devops/backups'
    }

    stages {
        stage('Resolve Environment') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'DEV') {
                        env.DB_NAME = 'flyway_dev'
                        env.DB_URL = 'jdbc:mysql://localhost:3306/flyway_dev?allowPublicKeyRetrieval=true&useSSL=false'
                        env.CRED_ID = 'mysql-flyway-dev'
                    }

                    if (params.ENVIRONMENT == 'QA') {
                        env.DB_NAME = 'flyway_qa'
                        env.DB_URL = 'jdbc:mysql://localhost:3306/flyway_qa?allowPublicKeyRetrieval=true&useSSL=false'
                        env.CRED_ID = 'mysql-flyway-qa'
                    }

                    if (params.ENVIRONMENT == 'PROD') {
                        env.DB_NAME = 'flyway_prod'
                        env.DB_URL = 'jdbc:mysql://localhost:3306/flyway_prod?allowPublicKeyRetrieval=true&useSSL=false'
                        env.CRED_ID = 'mysql-flyway-prod'
                    }

                    echo "Ambiente seleccionado: ${params.ENVIRONMENT}"
                    echo "Base de datos: ${env.DB_NAME}"
                }
            }
        }

        stage('Info Before') {
            steps {
                dir("${PROJECT_DIR}") {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${CRED_ID}",
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PASS'
                        )
                    ]) {
                        sh '''
                        $FLYWAY \
                          -url="$DB_URL" \
                          -user="$DB_USER" \
                          -password="$DB_PASS" \
                          -locations="filesystem:sql" \
                          info
                        '''
                    }
                }
            }
        }

        stage('Backup Database') {
            steps {
                dir("${PROJECT_DIR}") {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${CRED_ID}",
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PASS'
                        )
                    ]) {
                        sh '''
                        mkdir -p "$BACKUP_DIR"

                        TIMESTAMP=$(date +%Y%m%d_%H%M%S)
                        BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${ENVIRONMENT}_$TIMESTAMP.sql"

                        mysqldump \
                          --no-tablespaces \
                          -u"$DB_USER" \
                          -p"$DB_PASS" \
                          "$DB_NAME" \
                          > "$BACKUP_FILE"

                        echo "Backup generado:"
                        ls -lh "$BACKUP_FILE"
                        '''
                    }
                }
            }
        }

        stage('Production Approval') {
            when {
                expression { params.ENVIRONMENT == 'PROD' }
            }

            steps {
                input message: '¿Confirmas despliegue a PROD?', ok: 'Deploy'
            }
        }

        stage('Migrate') {
            steps {
                dir("${PROJECT_DIR}") {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${CRED_ID}",
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PASS'
                        )
                    ]) {
                        sh '''
                        $FLYWAY \
                          -url="$DB_URL" \
                          -user="$DB_USER" \
                          -password="$DB_PASS" \
                          -locations="filesystem:sql" \
                          migrate
                        '''
                    }
                }
            }
        }

        stage('Validate After') {
            steps {
                dir("${PROJECT_DIR}") {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${CRED_ID}",
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PASS'
                        )
                    ]) {
                        sh '''
                        $FLYWAY \
                          -url="$DB_URL" \
                          -user="$DB_USER" \
                          -password="$DB_PASS" \
                          -locations="filesystem:sql" \
                          validate
                        '''
                    }
                }
            }
        }

        stage('Info After') {
            steps {
                dir("${PROJECT_DIR}") {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${CRED_ID}",
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PASS'
                        )
                    ]) {
                        sh '''
                        $FLYWAY \
                          -url="$DB_URL" \
                          -user="$DB_USER" \
                          -password="$DB_PASS" \
                          -locations="filesystem:sql" \
                          info
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finalizado correctamente para ${params.ENVIRONMENT}"
        }

        failure {
            echo "Pipeline falló en ${params.ENVIRONMENT}. Revisar logs, backup y estado de Flyway."
        }
    }
}
