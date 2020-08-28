podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'golang', 
            image: 'golang:1.10-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helmmm', 
            image: 'ahmedcheibani/jenkins-slave-kubectl-helm:latest',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Checkout Source') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            sh "echo $commitId"
        }
        stage ('Build app ') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
            }
        }
        def repository
        stage ('Docker build && push ') {
            container ('docker') {
                def registryIp = sh(script: 'getent hosts registry.kube-system | awk \'{ print $1 ; exit }\'', returnStdout: true).trim()
                sh "echo $registryIp"
                repository = "ahmedcheibani/hello"
                withCredentials([usernamePassword(credentialsId: 'dockerhub_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
                {
                sh 'docker login --username="${USERNAME}" --password="${PASSWORD}"'
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
                }
            }
        }
        stage ('Helm Deploy') {
            container ('helmmm') {
               // sh "/helm init --client-only --skip-refresh"
                sh "helm upgrade hello hello -n web -i --wait --set image.repository=${repository},image.tag=${commitId}"
            }
        }
    }
}
