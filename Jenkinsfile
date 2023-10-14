node {
    stage('Build') {
        // Langkah-langkah untuk membangun proyek
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
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
}
