pipeline {
    agent any

    environment {
        imagename = "custom-nginx"
        registryCredential = 'anchor-ecr-credentials'
        dockerImage = ''
    }

    stages {
        // git에서 repository clone
        stage('Prepare') {
            steps {
                echo 'Clonning Repository'
                git url:'https://github.com/GoormAnchor/anchor-book-fe', branch:'Seyeon', credentialsId: 'anchor-repo-credentials';
            }
            post {
                success {
                    echo 'Successfully Cloned Repository'
                }
                failure {
                    error 'This pipeline stops here...'
                }
            }
        }

        // docker build
        stage('Bulid Docker') {
            steps {
                echo 'Bulid Docker'
                sh "docker build . -t 438282170065.dkr.ecr.ap-northeast-2.amazonaws.com/custom-nginx:${currentBuild.number}"
                sh "docker build . -t 438282170065.dkr.ecr.ap-northeast-2.amazonaws.com/custom-nginx:latest"
            }
            post {
                failure {
                    error 'This pipeline stops here...'
                }
            }
        }

        // docker push
        stage('Push Docker') {
            steps {
                echo 'Push Docker'
                docker.withRegistry('https://438282170065.dkr.ecr.ap-northeast-2.amazonaws.com/anchor-book-be', 'ecr:ap-northeast-2:anchor-ecr-credentials') {
                     dockerImage.push("${currentBuild.number}")
                    dockerImage.push("latest")
                }

            }
            post {
                failure {
                    error 'This pipeline stops here...'
                }   
            }
        }

        // k8s manifest update
         stage('K8S Manifest Update') {
        steps {
            git credentialsId: '{Credential ID}',
                url: 'https://github.com/best-branch/k8s-manifest.git',
                branch: 'master'

            sh "sed -i 's/my-app:.*\$/my-app:${currentBuild.number}/g' deployment.yaml"
            sh "git add deployment.yaml"
            sh "git commit -m '[UPDATE] my-app ${currentBuild.number} image versioning'"
            sshagent(credentials: ['{k8s-manifest repository credential ID}']) {
                sh "git remote set-url origin git@github.com:best-branch/k8s-manifest.git"
                sh "git push -u origin master"
             }
        }
        post {
                failure {
                  echo 'K8S Manifest Update failure !'
                }
                success {
                  echo 'K8S Manifest Update success !'
                }
    }
}