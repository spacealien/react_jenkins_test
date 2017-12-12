// @Library('deploy')
// import deploy

// def deployLib = new deploy()

node {
    def commitHash, commitHashShort, commitUrl, currentVersion
    def project = "navikt"
    def repo = ""
    def committer, committerEmail, changelog, pom, releaseVersion, nextVersion // metadata
    //def mvnHome = tool "maven-3.3.9"
    //def mvn = "${mvnHome}/bin/mvn"
    def bin = "usr/bin"
    def npm = "${bin}/npm"
    def node = "${bin}/node"
    def appConfig = "nais.yaml"
    //def dockerRepo = "docker.adeo.no:5000"
    def branch = "master"
    //def groupId = "nais"
    //def environment = 't1'
    //def zone = 'fss'
    d//ef namespace = 'default'

    stage("Checkout") {
        cleanWs()
        withEnv(['HTTPS_PROXY=http://webproxy-utvikler.nav.no:8088']) {
            sh(script: "git clone https://github.com/${project}/${repo}.git .")
        }
        commitHash = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
        commitHashShort = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        commitUrl = "https://github.com/${project}/${repo}/commit/${commitHash}"
        committer = sh(script: 'git log -1 --pretty=format:"%an"', returnStdout: true).trim()
        committerEmail = sh(script: 'git log -1 --pretty=format:"%ae"', returnStdout: true).trim()
        changelog = sh(script: 'git log `git describe --tags --abbrev=0`..HEAD --oneline', returnStdout: true)
        notifyGithub(project, repo, 'continuous-integration/jenkins', commitHash, 'pending', "Build #${env.BUILD_NUMBER} has started")
    }

    stage("Initialize") {
        //pom = readMavenPom file: 'pom.xml'
        //releaseVersion = pom.version.tokenize("-")[0]
    }

    stage("Install") {
        sh "${npm} install"
    }

    stage("Test") {
        sh "${npm} test"
    }

    stage("Build") {
        sh "${npm} build"
    }

    // stage("Build, test and install artifact") {
    //     sh "${mvn} clean install -Djava.io.tmpdir=/tmp/${repo} -B -e"

    //     sh "${npm} run test"
    //     sh "${npm} run build"
    // }

    // stage("Release") {
    //     sh "${mvn} versions:set -B -DnewVersion=${releaseVersion} -DgenerateBackupPoms=false"
    //     sh "${mvn} clean install -Djava.io.tmpdir=/tmp/${repo} -B -e"
      
    //   sh "docker build --build-arg version=${releaseVersion} --build-arg app_name=${repo} -t ${dockerRepo}/${repo}:${releaseVersion} ."
      
      
    //  sh "git commit -am \"set version to ${releaseVersion} (from Jenkins pipeline)\""
    //     withEnv(['HTTPS_PROXY=http://webproxy-utvikler.nav.no:8088']) {
    //         withCredentials([string(credentialsId: 'OAUTH_TOKEN', variable: 'token')]) {
    //             sh ("git push https://${token}:x-oauth-basic@github.com/${project}/${repo}.git master")
    //             sh ("git tag -a ${repo}-${releaseVersion} -m ${repo}-${releaseVersion}")
    //             sh ("git push https://${token}:x-oauth-basic@github.com/${project}/${repo}.git --tags")
    //         }
    //     }
    // }
    // stage("Publish artifact") {
      
    //     sh "docker push ${dockerRepo}/${repo}:${releaseVersion}"
    // }


    // stage("Deploy to t") {
    //     callback = "${env.BUILD_URL}input/Deploy/"
    //     deployLib.testCmd(releaseVersion)
    //     deployLib.testCmd(committer)

    //     def deploy = deployLib.deployNaisApp(repo, releaseVersion, environment, zone, namespace, callback, committer).key

    //     // Block and wait for the remote system to callback
    //     input id: 'Deploy', message: 'Waiting for remote system'


    // }

    // stage("Update project version") {
    //     nextVersion = (releaseVersion.toInteger() + 1) + "-SNAPSHOT"
    //     sh "${mvn} versions:set -B -DnewVersion=${nextVersion} -DgenerateBackupPoms=false"
    //     withEnv(['HTTPS_PROXY=http://webproxy-utvikler.nav.no:8088']) {
    //         withCredentials([string(credentialsId: 'OAUTH_TOKEN', variable: 'token')]) {
    //             sh "git commit -am \"updated to new dev-version ${nextVersion} after release by ${committer}\""
    //             sh ("git push https://${token}:x-oauth-basic@github.com/${project}/${repo}.git master")
    //         }
    //     }
    //     notifyGithub(project, repo, 'continuous-integration/jenkins', commitHash, 'success', "Build #${env.BUILD_NUMBER} has finished")
    // }
}

def notifyGithub(owner, repo, context, sha, state, description) {
    def postBody = [
            state: "${state}",
            context: "${context}",
            description: "${description}",
            target_url: "${env.BUILD_URL}"
    ]
    def postBodyString = groovy.json.JsonOutput.toJson(postBody)

    withEnv(['HTTPS_PROXY=http://webproxy-utvikler.nav.no:8088']) {
        withCredentials([string(credentialsId: 'OAUTH_TOKEN', variable: 'token')]) {
            sh """
                curl -H 'Authorization: token ${token}' \
                    -H 'Content-Type: application/json' \
                    -X POST \
                    -d '${postBodyString}' \
                    'https://api.github.com/repos/${owner}/${repo}/statuses/${sha}'
            """
        }
    }
}