pipeline {
    agent any

    parameters {
        choice(
            name: 'TARGET_ENV',
            choices: ['dev', 'test'],
            description: 'Select the target environment for this CI run'
        )
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    environment {
        APP_NAME = 'calculator'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code retrieved from GitHub.'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "Checking Python environment..."
                    python3 --version
                    echo "Application: $APP_NAME"
                '''
            }
        }

        stage('Run Configuration') {
            steps {
                echo "Selected environment: ${params.TARGET_ENV}"
                echo "Application: ${env.APP_NAME}"
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    echo "Running automated unit tests..."
                    python3 -m unittest discover -s test -v
                '''
            }
        }


        stage('Docker Build') {
    steps {
        script {
            env.IMAGE_TAG = "batch1-calculator:${env.BUILD_NUMBER}"
        }

        sh '''
            echo "Building Docker image..."
            docker build -t "$IMAGE_TAG" .
            echo "Docker image build completed."
        '''
    }
}

stage('Docker Image Validation') {
    steps {
        sh '''
            echo "Checking the Docker image..."
            docker image inspect "$IMAGE_TAG" >/dev/null
            echo "Docker image exists and is readable."
        '''
    }
}


        stage('Push Docker Image') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-credentials',
                usernameVariable: 'DOCKERHUB_USERNAME',
                passwordVariable: 'DOCKERHUB_TOKEN'
            )
        ]) {
            sh '''
                set +x

                IMAGE_REPO="$DOCKERHUB_USERNAME/batch1-calculator"
                VERSIONED_IMAGE="$IMAGE_REPO:$BUILD_NUMBER"

                echo "$DOCKERHUB_TOKEN" | docker login \
                    --username "$DOCKERHUB_USERNAME" \
                    --password-stdin

                docker tag "$IMAGE_TAG" "$VERSIONED_IMAGE"

                docker push "$VERSIONED_IMAGE"

                docker logout
            '''
        }
    }
}

        stage('Compile Check') {
            steps {
                sh '''
                    echo "Checking Python source syntax..."
                    python3 -m compileall -q app test
                    echo "Compile check passed."
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    mkdir -p dist

                    python3 - <<'PY'
import zipfile

files = [
    "app/calculator.py",
    "test/test_calculator.py"
]

with zipfile.ZipFile(
    "dist/calculator-package.zip",
    "w",
    zipfile.ZIP_DEFLATED
) as package:
    for file in files:
        package.write(file)

print("Application package created.")
PY

                    ls -lh dist/
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: 'dist/calculator-package.zip',
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'Application CI pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage and console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
