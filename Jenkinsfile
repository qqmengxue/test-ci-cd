podTemplate(label: 'jenkins-slave-pod',  containers: [
    /*
    containerTemplate(name: 'golang', image: 'registry.cn-hangzhou.aliyuncs.com/spacexnice/golang:1.6.3', ttyEnabled: true, command: 'cat'),
    */
    containerTemplate(name: 'docker', image: '172.28.5.75/chtest/mvn-git-kubectl-jnlp:3.5.19', command: '', ttyEnabled: false)
  ]
  ,volumes: [

      persistentVolumeClaim(mountPath: '/home/jenkins', claimName: 'jenkins', readOnly: false),

     hostPathVolume(hostPath: '/root/work/jenkins', mountPath: '/home/jenkins'),
     hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
     hostPathVolume(hostPath: '/tmp/', mountPath: '/tmp/'),
//     hostPathVolume(hostPath: '/root/.docker/', mountPath: '/root/.docker/'),
//     hostPathVolume(hostPath: '/usr/bin/kubectl', mountPath: '/usr/bin/kubectl'),
  ]){
node('jenkins-slave-pod') {
    stage "Chekck Env"
        sh "whoami"
        sh "date +%F"
    stage "chekckout SCM"
        checkout scm

        env.DOCKER_API_VERSION="1.13.1"
    stage "git checkout"
        sh "git rev-parse --short HEAD > commit-id"

        tag = readFile('commit-id').replace("\n", "").replace("\r", "")
        appName = "hello-kenzan"
        registryHost = "172.28.5.75/chtest/"
        imageName = "${registryHost}${appName}:${tag}"
        env.BUILDIMG=imageName

    stage "Build"
    
        sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
    
    stage "Push"

        sh "docker push ${imageName}"

    stage "Deploy"

        sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | /usr/bin/kubectl apply -f -"
        sh "/usr/bin/kubectl rollout status deployment/hello-kenzan"

}
}
