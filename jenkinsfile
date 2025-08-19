pipeline {
    agent any
    environment {
        SONAR_HOME = tool "sonar"
    }
    stages {
        stage("Clone Code") {
            steps {
                git url: "https://github.com/onkarlonkar9/wanderlust.git", branch: "main"
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("Sonar Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh """
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=wanderlust \
                        -Dsonar.projectName=wanderlust \
                        -Dsonar.sources=.
                    """
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML --out .', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage("Code Deploy") {
            steps {
                sh "docker-compose up -d"
            }
        }
    }
}
