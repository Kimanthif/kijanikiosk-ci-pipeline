pipeline {
    agent {
        docker {
            image 'node:18.17-alpine'
            args '-u root'
        }
    }

    environment {
        NODE_ENV = 'production'
        NEXUS_URL = 'http://172.17.0.1:8081'
        NEXUS_REPO = 'npm-hosted'
        PACKAGE_NAME = 'kijanikiosk-payments'
    }

    stages {

        stage('Lint') {
            steps {
                sh '''
                    npm install
                    npm run lint || { echo "Lint failed"; exit 1; }
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    # generate version = semver + git sha
                    GIT_SHA=$(git rev-parse --short HEAD)
                    CURRENT=$(node -p "require('./package.json').version")
                    NEW_VERSION="${CURRENT}-${GIT_SHA}"
                    npm version --no-git-tag-version "$NEW_VERSION"

                    # minimal build output
                    mkdir -p build
                    echo "Build output for version $NEW_VERSION" > build/output.txt
                '''
            }
        }

        stage('Verify') {
            parallel {
                stage('Test') {
                    steps {
                        sh '''
                            echo "Running tests..."
                            npm test
                        '''
                    }
                }

                stage('Security Audit') {
                    steps {
                        sh '''
                            echo "Running security audit..."
                            npm audit --audit-level=critical || true
                        '''
                    }
                }
            }
        }

        stage('Archive') {
            steps {
                sh 'tar -czf ${PACKAGE_NAME}.tgz build/'
                archiveArtifacts artifacts: '*.tgz', fingerprint: true
            }
        }

        stage('Publish') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        # Create temporary .npmrc
                        echo "//${NEXUS_URL#http://}/repository/${NEXUS_REPO}/:_authToken=${NEXUS_PASS}" > .npmrc
                        echo "registry=${NEXUS_URL}/repository/${NEXUS_REPO}/" >> .npmrc

                        npm pack
                        npm publish --registry=${NEXUS_URL}/repository/${NEXUS_REPO}/

                        rm .npmrc
                    '''
                }
            }
        }
    }

    post {
        always {
            junit 'test-results/**/*.xml' 2>/dev/null || true
            cleanWs()
        }
        success {
            echo "Artifact published at: ${NEXUS_URL}/repository/${NEXUS_REPO}/"
        }
        failure {
            echo "Pipeline failed — details in logs."
        }
        changed {
            echo "Build status changed since previous run."
        }
    }
}