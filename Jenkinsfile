pipeline {
  agent any
  options { timestamps() }

  // Using environment variables
  environment {
    APP_ENV = 'test'
    GREETING = 'Hello'
  }

  stages {
    // Create your first Pipeline (checkout)
    stage('Checkout') {
      steps {
        checkout scm
        echo "Repo checked out. Build #${env.BUILD_NUMBER} on branch ${env.BRANCH_NAME ?: 'N/A'}"
      }
    }

    // Run multiple steps (sequential)
    stage('Install dependencies') {
      steps {
        bat 'python --version'
        bat 'python -m pip install --upgrade pip'
        bat 'python -m pip install -r requirements.txt'
        bat 'echo %GREETING% from %APP_ENV% environment'
      }
    }

    // Run multiple steps (parallel)
    stage('Parallel checks') {
      parallel {
        stage('Unit tests') {
          steps {
            bat 'python -m pytest -q --junitxml=reports\\unit.xml'
          }
        }
        stage('Lint (demo)') {
          steps {
            bat 'python -c "print(\'lint ok (demo)\')"'
          }
        }
      }
    }

    // Recording artifacts
    stage('Package') {
      steps {
        // Make dist and zip app.py using PowerShell's Compress-Archive
        bat 'if not exist dist mkdir dist'
        powershell 'Compress-Archive -Path app.py -DestinationPath dist/app.zip -Force'
      }
    }

    // Deployment (manual gate)
    stage('Deploy to staging') {
      when { branch 'main' }
      steps {
        input message: 'Proceed with deployment to staging?'
        bat 'echo Pretend deploy: copying dist\\app.zip to staging...'
      }
    }
  }

  // Cleaning up and notifications
  post {
    always {
      junit 'reports\\*.xml'
      archiveArtifacts artifacts: 'dist\\**\\*.zip', fingerprint: true
      echo 'Cleaning workspace...'
      deleteDir()
    }
    success { echo '✅ SUCCESS: All stages passed.' }
    failure { echo '❌ FAILURE: See console log.' }
  }
}
