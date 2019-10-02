def verify_image(filename) {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
        sh '''
        #!/usr/env/bin bash
        docker run --rm \
        -e BRANCH_NAME \
        -v `pwd`:/home/tools/data \
        mojdigitalstudio/hmpps-packer-builder \
        bash -c 'ansible-galaxy install -r ansible/requirements.yml; USER=`whoami` packer validate ''' + filename + "'"
    }
}

def build_image(filename) {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
        sh """
        #!/usr/env/bin bash
        virtualenv venv_${filename}
        . venv_${filename}/bin/activate
        pip install -r requirements.txt
        python generate_metadata.py ${filename}
        deactivate
        rm -rf venv_${filename}

        set +x
        docker run --rm \
        -e BRANCH_NAME \
        -v `pwd`:/home/tools/data \
        mojdigitalstudio/hmpps-packer-builder \
        bash -c 'ansible-galaxy install -r ansible/requirements.yml; \
        PACKER_VERSION=`packer --version` USER=`whoami` packer build ${filename}'

        rm ./meta/${filename}_meta.json
        """
    }
}

pipeline {
    agent { label "jenkins_slave"}

    options {
        ansiColor('xterm')
    }

    triggers {
        cron(env.BRANCH_NAME=='master'? 'H 4 * * 7': '')
    }

    stages {
        stage ('Notify build started') {
            steps {
                slackSend(message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL.replace(':8080','')}|Open>)")
            }
        }

        stage('Verify PSN Proxy AMI') {
            parallel {
                stage('Verify CentOS PSN Proxy') { steps { script {verify_image('psn_proxy.json')}}}
            }
        }

        stage('Build PSN Proxy AMI') {
            parallel {
                stage('Build CentOS PSN Proxy AMI') { steps { script {build_image('psn_proxy.json')}}}
            }
        }
    }

    post {
        success {
            slackSend(message: "Build completed - ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'good')
        }
        failure {
            slackSend(message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'danger')
        }
    }
}

