pipeline {
    agent any
    stages {
        stage("Build") {
            environment {
                DB_HOST = credentials("laravel-host")
                DB_DATABASE = credentials("laravel-database")
                DB_USERNAME = credentials("laravel-user")
                DB_PASSWORD = credentials("laravel-password")
            }
            steps {
                sh 'php --version'
                sh 'composer install'
                sh 'composer --version'
                sh 'cp .env.example .env'
                sh 'echo DB_HOST=${DB_HOST} >> .env'
                sh 'echo DB_USERNAME=${DB_USERNAME} >> .env'
                sh 'echo DB_DATABASE=${DB_DATABASE} >> .env'
                sh 'echo DB_PASSWORD=${DB_PASSWORD} >> .env'
                sh 'php artisan key:generate'
                sh 'cp .env .env.testing'
                sh 'php artisan migrate'
            }
        }
        stage("Unit test") {
            steps {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            steps {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            steps {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            steps {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            environment {
                DOCKER_HUB_USERNAME = credentials("docker-hub-user")
                DOCKER_REPOSITORY = credentials("docker-reposirory")
                }   
            steps {
                sh "sudo docker build -t ${DOCKER_HUB_USERNAME}/${DOCKER_REPOSITORY}:${BUILD_NUMBER} ."
            }
        }
        
        stage("Docker push") {
            environment {
                DOCKER_USERNAME = credentials("docker-user")
                DOCKER_PASSWORD = credentials("docker-password")
            }
            steps {
                sh "sudo docker login --username ${DOCKER_USERNAME} --password ${DOCKER_PASSWORD}"
                sh "sudo docker push ab123cb/laravel:${BUILD_NUMBER}"
            }
        }
        stage("Deploy to staging") {
            steps {
                sh 'docker stop laravel8cd'
                sh "sudo docker run -d --rm -p 80:80 --name laravel8cd ab123cb/laravel:${BUILD_NUMBER} "
            }
        }
        stage("Acceptance test curl") {
            steps {
                sleep 20
                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
            }
        }
        
    }
}
