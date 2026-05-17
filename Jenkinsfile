pipeline {
agent any

```
stages {

    stage('Verify Workspace') {
        steps {
            sh '''
            echo "Current Workspace:"
            pwd

            echo "Files:"
            ls -R
            '''
        }
    }

    stage('Validate Flyway Migration') {
        steps {
            sh '''
            docker run --rm \
              --network flyway-poc_default \
              -v "$WORKSPACE/sql:/flyway/sql" \
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
              -v "$WORKSPACE/sql:/flyway/sql" \
              flyway/flyway \
              -url=jdbc:postgresql://postgres:5432/appdb \
              -user=flyway \
              -password=flyway123 \
              migrate
            '''
        }
    }
}
```

}
