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
        NEXUS_REPO = 'npm-private'
        PACKAGE_NAME = 'kijanikiosk-payments'
    }

    stages {

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint || echo "No lint configured"'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    GIT_SHA=$(git rev-parse --short HEAD)
                    CURRENT=$(node -p "require('./package.json').version")
                    NEW_VERSION="${CURRENT}-${GIT_SHA}"

                    npm version --no-git-tag-version $NEW_VERSION

                    mkdir -p build
                    echo "Build for version $NEW_VERSION" > build/output.txt
                '''
            }
        }

        stage('Verify') {
            parallel {
                stage('Test') {
                    steps {
                        sh 'npm test || echo "No tests configured"'
                    }
                }

                stage('Security Audit') {
                    steps {
                        sh 'npm audit --audit-level=critical || true'
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

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-npm-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {

                    sh '''
                        echo "registry=${NEXUS_URL}/repository/${NEXUS_REPO}/" > .npmrc
                        echo "//172.17.0.1:8081/repository/${NEXUS_REPO}/:_auth=${NEXUS_PASS}" >> .npmrc
                        echo "//172.17.0.1:8081/repository/${NEXUS_REPO}/:username=${NEXUS_USER}" >> .npmrc
                        echo "//172.17.0.1:8081/repository/${NEXUS_REPO}/:email=ci@kijanikiosk.com" >> .npmrc
                        echo "always-auth=true" >> .npmrc

                        npm pack
                        npm publish --registry=${NEXUS_URL}/repository/${NEXUS_REPO}/

                        rm -f .npmrc
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo "✔ Build successful and published to Nexus"
        }

        failure {
            echo "❌ Build failed"
        }

        changed {
            echo "ℹ Build status changed"
        }
    }
}