podTemplate(label: 'pod-kubernetes-client', 
    containers: [
        containerTemplate(
            name: 'kubernetes-client',
            image: 'alanhopkins/helm-kubectl-docker:1.00',
            ttyEnabled: true,
            command: 'cat')
      ],
      volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ]
) {
    node ('pod-kubernetes-client') {

    container('kubernetes-client') {
    echo "PATH = ${env.PATH}"

    def ret = sh(script: 'ls -l /usr/bin', returnStdout: true)
    println ret

    checkout scm

    env.DOCKER_API_VERSION="1.23"
    
    sh "git rev-parse --short HEAD > commit-id"

    // tag = readFile('commit-id').replace("\n", "").replace("\r", "")
    appName = "hello-kenzan"
    registryHost = "mycluster.icp:8500"
    // imageName = "${registryHost}/default/${appName}:${tag}"
    imageName = "${registryHost}/default/${appName}"
    env.BUILDIMG=imageName

    stage "Build"
    
        sh "/usr/bin/docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
    
    stage "Push"
 
        sh "docker login 'mycluster.icp:8500' -u 'admin' -p 'admin'"
        sh "docker push ${imageName}"
        sh "docker logout"

    stage "Deploy"

        sh "sed 's#mycluster.icp:8500/default/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
        sh "kubectl rollout status deployment/hello-kenzan"
    }
    }
}
