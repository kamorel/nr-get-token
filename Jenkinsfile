#!groovy

// --------------------
// Declarative Pipeline
// --------------------
pipeline {
  agent any

  environment {
    // Enable pipeline verbose debug output
    DEBUG_OUTPUT = true

    // Get projects/namespaces from config maps
    DEV_PROJECT = new File('/var/run/configs/ns/project.dev').getText('UTF-8').trim()
    TEST_PROJECT = new File('/var/run/configs/ns/project.test').getText('UTF-8').trim()
    PROD_PROJECT = new File('/var/run/configs/ns/project.prod').getText('UTF-8').trim()
    TOOLS_PROJECT = new File('/var/run/configs/ns/project.tools').getText('UTF-8').trim()

    // Get application config from config maps
    REPO_OWNER = new File('/var/run/configs/jobs/repo.owner').getText('UTF-8').trim()
    REPO_NAME = new File('/var/run/configs/jobs/repo.name').getText('UTF-8').trim()
    APP_NAME = new File('/var/run/configs/jobs/app.name').getText('UTF-8').trim()
    APP_DOMAIN = new File('/var/run/configs/jobs/app.domain').getText('UTF-8').trim()

    // JOB_NAME should be the pull request identifier (i.e. 'pr-5')
    JOB_NAME = "${JOB_BASE_NAME}".toLowerCase()

    // SOURCE_REPO_* references git repository resources
    SOURCE_REPO_RAW = "https://raw.githubusercontent.com/${REPO_OWNER}/${REPO_NAME}/master"
    SOURCE_REPO_REF="pull/${CHANGE_ID}/head"
    SOURCE_REPO_URL="https://github.com/${REPO_OWNER}/${REPO_NAME}.git"

    // HOST_ROUTE is the full domain route endpoint (ie. 'appname-pr-5-k8vopl-dev.pathfinder.gov.bc.ca')
    HOST_ROUTE = "${APP_NAME}-${JOB_NAME}-${DEV_PROJECT}.${APP_DOMAIN}"
  }

  stages {
    stage('Frontend') {
      steps {
        script {
          if (DEBUG_OUTPUT) {
            // Force OpenShift Plugin directives to be verbose
            openshift.logLevel(1)

            // Print project context and all environment variables
            echo "DEBUG - Using project: ${openshift.project()}"
            echo 'DEBUG - All pipeline environment variables:'
            echo sh(returnStdout: true, script: 'env')
          }
        }

        // Cancel any running builds in progress
        timeout(10) {
          echo 'Cancelling previous builds...'
          abortAllPreviousBuildInProgress(currentBuild)
        }

        script {
          openshift.withCluster() {
            openshift.withProject(TOOLS_PROJECT) {
              echo 'Creating Frontend BuildConfig...'
              def bcFrontend = openshift.process('-f',
                'openshift/frontend.bc.yaml',
                "REPO_NAME=${REPO_NAME}",
                "JOB_NAME=${JOB_NAME}",
                "SOURCE_REPO_URL=${SOURCE_REPO_URL}",
                "SOURCE_REPO_REF=${SOURCE_REPO_REF}"
              )

              echo 'Building Frontend...'
              openshift.apply(bcFrontend).narrow('bc').startBuild('-w').logs('-f')
              openshift.tag("${REPO_NAME}-frontend:latest",
                "${REPO_NAME}-frontend:${JOB_NAME}"
              )

              echo 'Creating Static Frontend BuildConfig...'
              def bcFrontendStatic = openshift.process('-f',
                'openshift/frontend-static.bc.yaml',
                "REPO_NAME=${REPO_NAME}",
                "JOB_NAME=${JOB_NAME}",
                "NAMESPACE=${TOOLS_PROJECT}"
              )

              echo 'Building Static Frontend...'
              openshift.apply(bcFrontendStatic).narrow('bc').startBuild('-w').logs('-f')
              openshift.tag("${REPO_NAME}-frontend-static:latest",
                "${REPO_NAME}-frontend-static:${JOB_NAME}"
              )
            }
          }
        }
      }
      post {
        cleanup {
          echo 'Cleanup Frontend BuildConfigs'
          script {
            try {
              openshift.withCluster() {
                openshift.withProject(TOOLS_PROJECT) {
                  openshift.selector('bc', "${REPO_NAME}-frontend-${JOB_NAME}").delete()
                  openshift.selector('bc', "${REPO_NAME}-frontend-static-${JOB_NAME}").delete()
                }
              }
            } catch (err) {
                echo err
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(DEV_PROJECT) {
              openshift.tag("${TOOLS_PROJECT}/nr-get-token-frontend-static:${JOB_NAME}", 'nr-get-token-frontend-static:${JOB_NAME}')
              def dcFrontend = openshift.process('-f',
                'openshift/frontend-static.dc.yaml',
                "REPO_NAME=${REPO_NAME}",
                "JOB_NAME=${JOB_NAME}",
                "NAMESPACE=${DEV_PROJECT}",
                "HOST_ROUTE=${HOST_ROUTE}"
              )
              openshift.apply(dcFrontend)
            }
          }
        }
      }
    }
  }
}
