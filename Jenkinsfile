pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    def sourceDir = 'sources'
                    def compiledResults = sh(script: "docker run --rm -v \$(pwd):/src -w /src/python:2-alpine python -m py_compile ${sourceDir}/add2vals.py ${sourceDir}/calc.py", returnStdout: true).trim()
                    stash name: 'compiled-results', includes: "${compiledResults}"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def testResults = sh(script: "docker run --rm -v \$(pwd):/src -w /src qnib/pytest py.test --verbose --junit-xml test-reports/results.xml ${sourceDir}/test_calc.py", returnStdout: true).trim()
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    dir('sources') {
                        unstash 'compiled-results'
                        def dockerImage = 'cdrx/pyinstaller-linux:python2'
                        sh "docker run --rm -v \$(pwd):/src -w /src ${dockerImage} pyinstaller -F add2vals.py"
                        archiveArtifacts artifacts: 'dist/*', allowEmptyArchive: true
                    }
                }
            }
        }
    }
}
