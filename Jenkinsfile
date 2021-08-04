pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Testing...'
                withGradle {
                    sh './gradlew clean test'
                }
            }
            post {
                always {
                    junit 'build/test-results/test/TEST-*.xml'
                    jacoco execPattern:'build/jacoco/*.exec'
                }
            }
        }
        stage('SonarQube Analysis') {
	    steps {
                withSonarQubeEnv('local') {
                    sh "./gradlew sonarqube"
                }
            }
        }
        stage('QA') {
            steps {
                withGradle {
                    sh './gradlew check'
                }
            }
            post {
                always {
                    recordIssues(
                        tools: [
                            pmdParser(pattern: 'build/reports/pmd/*.xml'),
                            spotBugs(pattern: 'build/reports/spotbugs/*.xml', useRankAsPriority: true)
                        ]
                    )
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'docker-compose build'
            }
        }
        stage('Security') {
            steps {
                echo 'Security analysis...'
                sh 'trivy image --format=json --output=trivy-image.json hello-spring-testing:latest'
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        aggregatingResults: true,
                        tool: trivy(pattern: 'trivy-*.json')
                    )
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
