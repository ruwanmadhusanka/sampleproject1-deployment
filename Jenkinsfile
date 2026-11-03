def CAPPS = [
  dev:     '/opt/wso2/mi-dev/repository/deployment/server/carbonapps',
  staging: '/opt/wso2/mi-staging/repository/deployment/server/carbonapps',
  prod:    '/opt/wso2/mi-prod/repository/deployment/server/carbonapps',
]
pipeline {
  agent any
  triggers { githubPush() }
  options { disableConcurrentBuilds() }
  environment {
    DEPLOY_USER = 'ruwanw999'
    DEPLOY_HOST = 'localhost'
    NEXUS_BASE  = 'http://localhost:8081/repository/integration'
  }
  stages {
    stage('Resolve environment') {
      steps {
        script {
          def branch = env.GIT_BRANCH.replaceFirst(/^origin\//, '')
          if (!CAPPS.containsKey(branch)) { error "No environment mapped to branch '${branch}'" }
          env.TARGET_ENV = branch
          env.CAPPS_PATH = CAPPS[branch]
        }
      }
    }
    stage('Fetch CApps from Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh '''
            set -eu
            rm -rf fetched && mkdir fetched
            grep -h '^- ' descriptors/*.yaml > capps.list || { echo "No CApp entries in descriptors/"; exit 1; }
            while read -r _ group artifact version; do
              url="$NEXUS_BASE/$group/$artifact/$version/$artifact-$version.car"
              echo "Fetching $url"
              curl -fsS -u "$NEXUS_USER:$NEXUS_PASS" --connect-timeout 20 -o "fetched/$artifact-$version.car" "$url"
            done < capps.list
          '''
        }
      }
    }
    stage('Deploy to MI') {
      steps {
        sh '''
          set -eu
          ssh -o StrictHostKeyChecking=accept-new "$DEPLOY_USER@$DEPLOY_HOST" "rm -f '$CAPPS_PATH'/*.car"
          scp fetched/*.car "$DEPLOY_USER@$DEPLOY_HOST:$CAPPS_PATH/"
          echo "Deployed to $TARGET_ENV:"; ls fetched
        '''
      }
    }
  }
}