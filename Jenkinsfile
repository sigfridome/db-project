pipeline {
    agent any

    environment {
        FLYWAY = '/opt/devops/tools/flyway/flyway'
        DB_URL = 'jdbc:mysql://localhost:3306/flyway_lab?allowPublicKeyRetrieval=true&useSSL=false'
        DB_USER = 'flyway_user'
        DB_PASS = 'Sefin2026_'
    }

    stages {

        stage('Validate') {
            steps {
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

        stage('Info') {
            steps {
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

        stage('Migrate') {
            steps {
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
