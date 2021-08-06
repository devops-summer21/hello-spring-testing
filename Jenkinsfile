#!/usr/bin/env groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Testing...'
                withGradle {
                    sh './gradlew clean test pitest'
                }
            }
            post {
                always {
                    junit 'build/test-results/test/TEST-*.xml'
                    jacoco execPattern:'build/jacoco/*.exec'
                    recordIssues(enabledForFailure: true, tool: pit(pattern:"build/reports/pitest/**/*.xml"))
                }
            }
        }
	stage('Analysis') {
		parallel {
			stage('SonarQube Analysis') {
			    when { expression { false } }
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
        stage('Publish') {
            steps {
                tag 'docker tag 10.250.0.6:5050/codehead/hello-spring/hello-spring-testing:latest 10.250.0.6:5050/codehead/hello-spring/hello-spring-testing:TESTING-1.0.${BUILD_NUMBER}'
                withDockerRegistry([url:'10.250.0.6:5050', credentialsId:'dockerCLI' ]) {
                    sh 'docker push 10.250.0.6:5050/codehead/hello-spring/hello-spring-testing:latest'
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
