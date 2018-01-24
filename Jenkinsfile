podTemplate(label: 'pod-kubernetes-client', 
    containers: [
        containerTemplate(
            name: 'kubernetes-client',
            image: 'alanhopkins/helm-kubectl-docker:latest',
            ttyEnabled: true,
            command: 'cat'),
        containerTemplate(
            name: 'docker',
            privileged: true,
            image: 'alanhopkins/docker-client:latest',
            ttyEnabled: true,
            command: 'cat')
      ]
) 
{
    node ('pod-kubernetes-client') {

       container('kubernetes-client') {
          echo "PATH = ${env.PATH}"

          def ret = sh(script: 'ls -l /usr/bin', returnStdout: true)
          println ret

          def id = sh(script: 'id', returnStdout: true)
          println id

          checkout scm

          env.DOCKER_API_VERSION="1.23"
    
          //sh "git rev-parse --short HEAD > commit-id"

          // tag = readFile('commit-id').replace("\n", "").replace("\r", "")
          appName = "hello-kenzan"
          registryHost = "mycluster.icp:8500"
          // imageName = "${registryHost}/default/${appName}:${tag}"
          imageName = "${registryHost}/default/${appName}"
          env.BUILDIMG=imageName
       }
       container('docker') {
          //stage "Build" {
    
             def id = sh(script: 'id', returnStdout: true)
             println id

             sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
          //}
          //stage "Push" {
 
             sh "docker login 'mycluster.icp:8500' -u 'admin' -p 'admin'"
             sh "docker push ${imageName}"
             sh "docker logout"
         //}
      }
      container('kubernetes-client') {
         //stage "Deploy" {

             sh "sed 's#mycluster.icp:8500/default/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
             sh "kubectl rollout status deployment/hello-kenzan"
         //}
      }
   }
}
