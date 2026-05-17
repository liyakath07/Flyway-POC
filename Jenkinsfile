pipeline {
    agent any

    stages {

        stage('Validate Flyway Migration') {
            steps {
                sh '''
                docker run --rm \
                  --network flyway-poc_default \
                  -v $(pwd)/sql:/flyway/sql \
                  flyway/flyway \
                  -url=jdbc:postgresql://postgres:5432/appdb \
                  -user=flyway \
                  -password=flyway123 \
                  validate
                '''
            }
        }

        stage('Execute Flyway Migration') {
            steps {
                sh '''
                docker run --rm \
                  --network flyway-poc_default \
                  -v $(pwd)/sql:/flyway/sql \
                  flyway/flyway \
                  -url=jdbc:postgresql://postgres:5432/appdb \
                  -user=flyway \
                  -password=flyway123 \
                  migrate
                '''
            }
        }
    }
}
