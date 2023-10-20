pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
               stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed', parameters: [choice(choices: ['Proceed', 'Abort'], description: 'Pilih tindakan', name: 'ACTION')])
                    if (userInput == 'Abort') {
                        error 'Pipeline dihentikan oleh pengguna'
                    }
                }
            }
        }

        stage('Deploy') {
            agent any
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
                script {
                    // Menjeda eksekusi pipeline selama 1 menit (60 detik)
                    echo 'Menjeda eksekusi pipeline selama 1 menit...'
                    sleep time: 60, unit: 'SECONDS'
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}
