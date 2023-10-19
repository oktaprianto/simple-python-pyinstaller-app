node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }

    stage('Deploy') {
        agent any
        environment {
            VOLUME = "${WORKSPACE}/sources:/src"
            IMAGE = 'cdrx/pyinstaller-linux:python2'
        }
        steps {
            dir(path: env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
            }
        }
        post {
            success {
                archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", allowEmptyArchive: true
                sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
            }
        }
    }
}
