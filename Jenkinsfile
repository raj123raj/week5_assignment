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
                
                // Install PHP dependencies -> creates vendor/autoload.php
                bat "%COMPOSER_PATH% install --no-progress --no-interaction"
				
            }
        }

      stage('Environment') {
    steps {
        bat """
        copy /Y .env.example .env
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
                    if exist exclude.txt (
                        xcopy /E /I /Y /H . "%LOCAL_DEPLOYPATH%" /exclude:exclude.txt
                    ) else (
                        xcopy /E /I /Y /H . "%LOCAL_DEPLOYPATH%"
                    )
                """


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
