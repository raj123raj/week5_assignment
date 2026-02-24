pipeline {
    agent any

    environment {
        PHP_PATH       = 'D:\\Priya\\softwares\\php\\php.exe'
        COMPOSER_PATH  = 'D:\\Priya\\softwares\\composer\\composer.bat'
        LOCAL_DEPLOYPATH = 'D:\\Priya\\my-laravel-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-raj123raj',
                    url: 'https://github.com/raj123raj/week5_assignment'
            }
        }

        stage('Dependencies') {
            steps {
                // Verify composer.json exists in workspace
                bat 'dir composer.json'                
                bat 'dir vendor || echo NO VENDOR YET'
                // Install PHP dependencies -> creates vendor/autoload.php
                bat "%COMPOSER_PATH% install --no-progress --no-interaction"
				bat 'dir vendor\\autoload.php'  // ✅ Exists
            }
        }

        stage('Environment') {
            steps {
                bat """
                if not exist .env copy /Y.env.example .env
                "%PHP_PATH%" artisan key:generate --force
                """
            }
        }

        stage('Test DB') {
            steps {
                bat """
                if exist database\\database.sqlite del database\\database.sqlite
                "%PHP_PATH%" artisan migrate:fresh --seed
                """
            }
        }

        stage('Test') {
            steps {
                bat "\"%PHP_PATH%\" artisan test"
            }
        }

        stage('Deploy Local') {
            steps {
                bat """
                xcopy /E /I /Y /H . "%LOCAL_DEPLOYPATH%" /exclude:exclude.txt
                """
            }
        }

        stage('Deploy Server') {
            steps {
                sshagent(['ssh-deploy-key']) {
                    // Note: this ssh+remote script assumes composer/php are on PATH on the server
                    bat """
                    ssh -o StrictHostKeyChecking=no ubuntu@your-server \"
                    cd /var/www/my-app &&
                    git pull origin main &&
                    composer install --no-dev --no-interaction &&
                    php artisan migrate --force &&
                    php artisan optimize &&
                    sudo systemctl reload nginx php8.3-fpm \"
                    """
                }
            }
        }
    }

    post {
        always {
            // Only if you really produce JUnit XML into storage/logs; otherwise remove this.
            junit 'storage/logs/*.xml'
            cleanWs()
        }
    }
}
