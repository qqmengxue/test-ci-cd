podTemplate(label: 'jenkins-slave-pod',  containers: [
    /*
    containerTemplate(name: 'golang', image: 'registry.cn-hangzhou.aliyuncs.com/spacexnice/golang:1.6.3', ttyEnabled: true, command: 'cat'),
    */
    containerTemplate(name: 'docker', image: 'registry.cn-hangzhou.aliyuncs.com/spacexnice/jenkins-slave:latest', command: '', ttyEnabled: false)
  ]
  ,volumes: [
/*
      persistentVolumeClaim(mountPath: '/home/jenkins', claimName: 'jenkins', readOnly: false),
*/
     hostPathVolume(hostPath: '/root/work/jenkins', mountPath: '/home/jenkins'),
     hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
     hostPathVolume(hostPath: '/tmp/', mountPath: '/tmp/'),
  ]){
node('jenkins-slave-pod') {

    checkout scm

    env.DOCKER_API_VERSION="1.13.1"
    
    sh "git rev-parse --short HEAD > commit-id"

    tag = readFile('commit-id').replace("\n", "").replace("\r", "")
    appName = "hello-kenzan"
    registryHost = "172.28.5.75/chtest/"
    imageName = "${registryHost}${appName}:${tag}"
    env.BUILDIMG=imageName

    stage "Build"
    
        sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
    
    stage "Push"
    container('docker') {
        sh "docker push ${imageName}"
    }
    stage "Deploy"
    container('docker') {
        sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
        sh "kubectl rollout status deployment/hello-kenzan"
    }
}
}
