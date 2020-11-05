node {
  stage("Build") {
    withPod {
      node('random') {
        def tag = "${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
          def service = "market-data:${tag}"

          checkout scm

          container('docker') {
            stage('Build') {
              sh("cd chapter-10/market-data/ && docker build -t ${service} .")
            }

            stage('Test') {
              try {
                sh("docker run -v `pwd`/chapter-10/market-data:/workspace --rm ${service} python setup.py test")
              } finally {
                step([$class: 'JUnitResultArchiver', testResults: 'chapter-10/market-data/results.xml'])
              }
            }

            def tagToDeploy = "jdk2588/${service}"

              stage('Publish') {
                withDockerRegistry(registry: [credentialsId: 'docker']) {
                  sh("docker tag ${service} ${tagToDeploy}")
                    sh("docker push ${tagToDeploy}")
                }
              }

            stage('Deploy') {
              sh("sed -i.bak 's#BUILD_TAG#${tagToDeploy}#' ./chapter-10/deploy/staging/*.yml")
                container('kubectl') {
                  sh("kubectl --namespace=staging apply -f chapter-10/deploy/staging/ --kubeconfig=/tmp/config")
                }
            }
          }
      }
    }
  }
}

def withPod(body) {
  podTemplate(label: 'box', serviceAccount: 'jenkins', containers: [
      containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
      containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) { body() }
}
