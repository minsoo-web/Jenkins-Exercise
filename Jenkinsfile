pipeline {
    agent any

    options {
        ansiColor('xterm')
    }

    parameters {
        choice(name:'ID', choices:['root','user0','QA','ALL'], description:'ID')
        choice(name:'build_target', choices:['IRIS-E2E','IRIS-E2E-SAAS','SAMPLE-E2E'], description:'Build_target')
        string(name:'menu_target', defaultValue:'ALL', description:'build for what')
        string(name:'container_number',defaultValue:"$BUILD_NUMBER", description:'build_number for container name')
    }

    environment {
        SAAS_CONTAINER_NAME = "new-iris-e2e-saas-${params.container_number}"
        BASE_IMAGE_NAME = "e2e-base-image:latest"
    }

    stages {
        stage('BUILD CONTAINER') {
            // 빌드를 하기 전 테스트를 진행할 side 파일들을 파라미터에 맞게 수정합니다.
            steps {
                dir ('IRIS-E2E-SAAS')  {
                        git branch: 'master',
                        credentialsId: '8049ffe0-f4fb-4bfe-ab97-574e07244a32',
                        url: 'https://github.com/mobigen/IRIS-E2E-SAAS.git'

                        sh"""
                        # side file 만 추출
                        python3 ../docker_build.py
                        """
                        sh"""
                        ls
                        cd dist
                        ls
                        """
                }
            }
        }

        stage('BUILD IMAGE') { 
            steps {
                script {
                    if (build_target == "SAMPLE-E2E"){
                        sh"""
                        docker run -itd --privileged -p 4444:4444 --name ${SAAS_CONTAINER_NAME} ${BASE_IMAGE_NAME}
                        docker cp dist/IRIS-E2E-SAAS ${SAAS_CONTAINER_NAME}:/root/IRIS-E2E/IRIS-E2E-SAAS
                        """
                    }
                }
            }
        }

        stage('BUILD JOB') {
            // 전처리가 끝난 다음 job을 전달합니다.
            when {
                triggeredBy "UserCause"
            }
            steps {
                build(
                    job: "$params.build_target",
                    wait: true,
                    parameters: [
                        string(
                            name: 'AUTO', 
                            value: 'NOT_AUTO'
                        )
                    ]
                )
                echo "$params.ID"
                echo "$params.build_target"
                echo "$params.menu_target"
            }   
        }

        stage('AFTER BUILD JOB') {
            steps {
                sh "ls"
            }
        }
    }
}

