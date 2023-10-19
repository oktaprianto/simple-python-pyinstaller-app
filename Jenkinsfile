node {
   stage('Build') {
        def BUILD_DIR = pwd()
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    stage('Test') {
        // Langkah-langkah untuk menjalankan pengujian
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        // Langkah-langkah pasca pengujian
        junit 'test-reports/results.xml'
    }

      stage('Deliver') {
        def BUILD_DIR = pwd()
        def VOLUME = "${BUILD_DIR}/sources:/src"
        def IMAGE = 'cdrx/pyinstaller-linux:python2'

        dir("${BUILD_DIR}") {
            unstash 'compiled-results'
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }

        archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
    }
}