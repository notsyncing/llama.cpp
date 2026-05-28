pipeline {
    agent any

    parameters {
        string(
            name: 'BUILD_HTTP_PROXY',
            description: 'Build http proxy',
            defaultValue: ''
        )
        string(
            name: 'BRANCH',
            defaultValue: 'sycl-playground',
            description: 'Branch to build'
        )
        string(
            name: 'REGISTRY',
            defaultValue: '10.201.0.4:5000',
            description: 'Docker registry URL'
        )
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Image tag'
        )
    }

    environment {
        IMAGE_NAME = "${params.REGISTRY}/llama-sycl-xmx"
        IMAGE_TAG = "${params.IMAGE_TAG}"
        BUILD_HTTP_PROXY = "${params.BUILD_HTTP_PROXY}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${params.BRANCH}"]],
                    userRemoteConfigs: [[url: 'https://github.com/notsyncing/llama.cpp.git']],
                    extensions: [[$class: 'CleanCheckout']]
                ])
            }
        }

        stage('Build Docker') {
            steps {
                sh '''
                    docker build \\
                        --build-arg https_proxy=${BUILD_HTTP_PROXY} \\
                        --build-arg no_proxy="127.0.0.1,localhost,/run/buildkit/buildkitd.sock" \\
                        --build-arg GGML_SYCL_F16=ON \\
                        --build-arg GGML_SYCL_GRAPH=ON \\
                        --target full \\
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \\
                        -f .devops/intel.Dockerfile \\
                        .
                '''
            }
        }

        stage('Push') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
