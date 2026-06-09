pipeline {
    agent {
        label 'linux'
    }

    environment {
        FLYWAY = '/opt/devops/tools/flyway/flyway'
    }

    stages {

        stage('Deploy DEV') {
            steps {
                script {
                    deployDatabase(
                        'DEV',
                        'POCDB',
                        'mysql-flyway-dev' // Nota: Si creas una credencial nueva en Jenkins con los datos de Oracle, cambia este nombre aquí
                    )
                }
            }
        }

        stage('QA Approval') {
            steps {
                input message: '¿Desplegar a QA?', ok: 'Deploy QA'
            }
        }

        stage('Deploy QA') {
            steps {
                script {
                    deployDatabase(
                        'QA',
                        'POCDB',
                        'mysql-flyway-qa'
                    )
                }
            }
        }

        stage('PROD Approval') {
            steps {
                input message: '¿Desplegar a PROD?', ok: 'Deploy PROD'
            }
        }

        stage('Deploy PROD') {
            steps {
                script {
                    deployDatabase(
                        'PROD',
                        'POCDB',
                        'mysql-flyway-prod'
                    )
                }
            }
        }
    }
}

def deployDatabase(envName, dbName, credentialId) {

    withCredentials([
        usernamePassword(
            credentialsId: credentialId,
            usernameVariable: 'DB_USER',
            passwordVariable: 'DB_PASS'
        )
    ]) {

        sh """
        echo '=============================='
        echo 'Deploy Flyway Ambiente: ${envName}'
        echo 'Base de datos: ${dbName}'
        echo '=============================='

        DB_HOST="172.16.88.60"
        DB_PORT="1521"
        DB_SERVICE="POCDB"

        JDBC_URL="jdbc:oracle:thin:@//\${DB_HOST}:\${DB_PORT}/\${DB_SERVICE}"

        echo 'Consultando estado actual de las migraciones en Oracle...'
        \$FLYWAY \
          -url="\${JDBC_URL}" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          info

        echo 'Aplicando nuevas migraciones V en la base corporativa...'
        \$FLYWAY \
          -url="\${JDBC_URL}" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          migrate

        echo 'Validando integridad del historial...'
        \$FLYWAY \
          -url="\${JDBC_URL}" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          validate
        """
    }
}
