pipeline {
  agent any
  options { timestamps() }

  environment {
    APP_ENV = 'test'
    GREETING = 'Hello'
    VENV = "${WORKSPACE}/.venv"
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
          # create / reuse virtualenv (requires python3-venv installed on the node/container)
          python3 -m venv "$VENV"
          "$VENV/bin/python" -V
          "$VENV/bin/python" -m pip install --upgrade pip
          "$VENV/bin/python" -m pip install -r requirements.txt
          echo "$GREETING from ${APP_ENV} environment"
        '''
      }
    }

    stage('Parallel checks') {
      parallel {
        stage('Unit tests') {
          steps {
            sh '''
              set -eux
              mkdir -p reports
              "$VENV/bin/python" -m pytest -q --junitxml=reports/unit.xml
            '''
          }
        }
        stage('Lint (demo)') {
          steps {
            sh '''
              set -eux
              "$VENV/bin/python" - <<'PY'
print("lint ok (demo)")
PY
            '''
          }
        }
      }
    }

    stage('Package') {
      steps {
        sh '''
          set -eux
          mkdir -p dist
          "$VENV/bin/python" - <<'PY'
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
