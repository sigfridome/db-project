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
                        'flyway_dev',
                        'mysql-flyway-dev'
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
                        'flyway_qa',
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
                        'flyway_prod',
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
        echo 'Deploy ambiente: ${envName}'
        echo 'Base de datos: ${dbName}'
        echo '=============================='

        mkdir -p /opt/devops/backups

        TIMESTAMP=\$(date +%Y%m%d_%H%M%S)

        mysqldump \
          --no-tablespaces \
          -u\$DB_USER \
          -p\$DB_PASS \
          ${dbName} \
          > /opt/devops/backups/${dbName}_${envName}_\$TIMESTAMP.sql

        echo 'Backup generado'

        $FLYWAY \
          -url="jdbc:mysql://localhost:3306/${dbName}?allowPublicKeyRetrieval=true&useSSL=false" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          info

        $FLYWAY \
          -url="jdbc:mysql://localhost:3306/${dbName}?allowPublicKeyRetrieval=true&useSSL=false" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          migrate

        $FLYWAY \
          -url="jdbc:mysql://localhost:3306/${dbName}?allowPublicKeyRetrieval=true&useSSL=false" \
          -user="\$DB_USER" \
          -password="\$DB_PASS" \
          -locations="filesystem:sql" \
          validate
        """
    }
}
