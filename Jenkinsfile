// get projects/namespaces from config maps
def devProject = new File('/var/run/configs/ns/project.dev').getText('UTF-8').trim()
def testProject = new File('/var/run/configs/ns/project.test').getText('UTF-8').trim()
def prodProject = new File('/var/run/configs/ns/project.prod').getText('UTF-8').trim()
def toolsProject = new File('/var/run/configs/ns/project.tools').getText('UTF-8').trim()

// get application config from config maps
def repoOwner = new File('/var/run/configs/jobs/repo.owner').getText('UTF-8').trim()
def appRepo = new File('/var/run/configs/jobs/repo.name').getText('UTF-8').trim()
def appName = new File('/var/run/configs/jobs/app.name').getText('UTF-8').trim()
def appDomain = new File('/var/run/configs/jobs/app.domain').getText('UTF-8').trim()

def nameSelector = "${appName}"
def label = "'app-name'=${appName}"
def rawRepoBase = ""
def devDomain = ""

def doEcho = true

pipeline {
    agent any

    stages {

        stage('preamble') {
            steps {
                echo "Print out all environment variables in this pipeline."
                echo sh(returnStdout: true, script: 'env')
                script {
                    openshift.withCluster() {
                        openshift.withProject(devProject) {

                            // fill in some of the globals here...
                            rawRepoBase = "https://raw.githubusercontent.com/${repoOwner}/${appRepo}/master"
                            devDomain = "${nameSelector}-${devProject}.${appDomain}"

                            if (doEcho) {
                                echo "Using project: ${openshift.project()}"
                                echo "----- Environment -----"
                                echo "   Git"
                                echo "      BRANCH_NAME = ${BRANCH_NAME}"
                                echo "      GIT_BRANCH = ${GIT_BRANCH}"
                                echo "      GIT_COMMIT = ${GIT_COMMIT}"

                                echo "   NS Config"
                                echo "      devProject = ${devProject}"
                                echo "      testProject = ${testProject}"
                                echo "      prodProject = ${prodProject}"
                                echo "      toolsProject = ${toolsProject}"

                                echo "   App ConfigMap"
                                echo "      repoOwner = ${repoOwner}"
                                echo "      appRepo = ${appRepo}"
                                echo "      appName = ${appName}"
                                echo "      appDomain = ${appDomain}"

                                echo "   Global Variables"
                                echo "   nameSelector = ${nameSelector}"
                                echo "   label = ${label}"
                                echo "   devDomain = ${devDomain}"
                                echo "   rawRepoBase = ${rawRepoBase}"
                            }

                        }
                    }
                }
            }
        }

    }
}
