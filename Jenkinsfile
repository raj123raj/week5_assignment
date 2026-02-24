pipeline {
    agent any
    environment {
        PHP_PATH = 'D:\\\\Priya\\\\softwares\\\\php\\\\php.exe'
        COMPOSER_PATH = 'D:\\\\Priya\\\\softwares\\\\composer\\\\composer.bat'
        LOCAL_DEPLOYPATH = 'D:\\\\Priya\\\\my-laravel-app'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-raj123raj',
                    url: 'https://github.com/raj123raj/my-laravel-app'
            }
        }

        stage('Dependencies') {
            steps {
			 bat 'dir composer.json'  // Verify in workspace
        bat 'dir vendor || echo NO VENDOR YET'
                bat "${COMPOSER_PATH} install --no-progress --no-interaction"
				bat 'dir vendor\\autoload.php'  // ✅ Exists
            }
        }
        stage('Environment') {
            steps {
                bat 'copy .env.example .env'
                bat "${PHP_PATH} artisan key:generate --force"
            }
        }		
		
        stage('Test DB') {
            steps {
                bat """
                if exist database\\database.sqlite del database\\database.sqlite
                ${PHP_PATH} artisan migrate:fresh --seed
                """
            }
        }
        stage('Test') {
            steps {
                bat "${PHP_PATH} artisan test"
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
                    bat """
                    ssh -o StrictHostKeyChecking=no ubuntu@your-server \"
                    cd /var/www/my-app &&
                    git pull origin main &&
                    composer install --no-dev &&
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
            junit 'storage/logs/*.xml'
            cleanWs()
        }
    }
}
