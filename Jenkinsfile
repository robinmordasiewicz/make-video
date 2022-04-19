pipeline {
  options {
    disableConcurrentBuilds()
    skipDefaultCheckout(true)
  }
//  triggers {
//    upstream(upstreamProjects: "sphinx-theme,f5-cnf-docs", threshold: hudson.model.Result.SUCCESS)
//  }
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: mlt
            image: robinhoodis/mlt:latest
            imagePullPolicy: Always
            command:
            - cat
            tty: true
        '''
    }
  }
  stages {
    stage('INIT') {
      steps {
        cleanWs()
        checkout scm
      }
    }
    stage('checkout docs') {
      steps {
        sh 'mkdir -p docs'
        dir ( 'docs' ) {
          git branch: 'main', url: 'https://github.com/robinmordasiewicz/f5-cnf-lab.git'
        }
      }
    }
    stage('make video') {
      steps {
        dir ( 'docs' ) {
          container('mlt') {
            sh 'ffmpeg -f concat -filter_complex xfade=transition=slideleft:duration=5:offset=0 -i join.txt -c copy output.mov'
          }
        }
      }
    }
    stage('Commit new VERSION') {
      steps {
        dir ( 'docs' ) {
          sh 'git config user.email "robin@mordasiewicz.com"'
          sh 'git config user.name "Robin Mordasiewicz"'
          // sh 'git add -u'
          // sh 'git diff --quiet && git diff --staged --quiet || git commit -m "`cat VERSION`"'
          sh 'git add . && git diff --staged --quiet || git commit -m "new movie"'
          withCredentials([gitUsernamePassword(credentialsId: 'github-pat', gitToolName: 'git')]) {
            // sh 'git diff --quiet && git diff --staged --quiet || git push origin HEAD:main'
            // sh 'git diff --quiet HEAD || git push origin HEAD:main'
            sh 'git push origin HEAD:main'
          }
        }
      }
    }
  }
  post {
    always {
      cleanWs(cleanWhenNotBuilt: false,
            deleteDirs: true,
            disableDeferredWipeout: true,
            notFailBuild: true,
            patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                       [pattern: '.propsfile', type: 'EXCLUDE']])
    }
  }
}
