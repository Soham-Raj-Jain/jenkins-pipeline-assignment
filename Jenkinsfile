pipeline {
  agent any
  options { timestamps() }

  environment {
    APP_ENV = 'test'
    GREETING = 'Hello'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        echo "Repo checked out. Build #${env.BUILD_NUMBER}"
      }
    }

    stage('Install dependencies') {
      steps {
        sh '''
          set -eux
          if ! command -v python3 >/dev/null 2>&1; then
            echo "Python3 is required on this Jenkins node." >&2
            exit 1
          fi
          python3 -V
          python3 -m pip install --upgrade pip
          python3 -m pip install -r requirements.txt
          echo "$GREETING from ${APP_ENV} environment"
        '''
      }
    }

    stage('Parallel checks') {
      parallel {
        stage('Unit tests') {
          steps { sh 'python3 -m pytest -q --junitxml=reports/unit.xml' }
        }
        stage('Lint (demo)') {
          steps { sh 'python3 - <<EOF\nprint("lint ok (demo)")\nEOF' }
        }
      }
    }

    stage('Package') {
      steps {
        sh '''
          mkdir -p dist
          python3 - <<'PY'
import zipfile
zf = zipfile.ZipFile('dist/app.zip','w')
zf.write('app.py')
zf.close()
PY
        '''
      }
    }

    stage('Deploy to staging') {
      when { branch 'main' }
      steps {
        input message: 'Proceed with deployment to staging?'
        sh 'echo "Pretend deploy: copying dist/app.zip to staging..."'
      }
    }
  }

  post {
    always {
      junit 'reports/*.xml'
      archiveArtifacts artifacts: 'dist/**/*.zip', fingerprint: true
      echo 'Cleaning workspace...'
      deleteDir()
    }
    success { echo '✅ SUCCESS: All stages passed.' }
    failure { echo '❌ FAILURE: See console log.' }
  }
}
