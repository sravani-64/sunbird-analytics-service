node('build-slave') {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"
        ansiColor('xterm') {
            stage('Checkout') {
                if (!env.hub_org) {
                    println(ANSI_BOLD + ANSI_RED + "Uh Oh! Please set a Jenkins environment variable named hub_org with value as registery/sunbidrded" + ANSI_NORMAL)
                    error 'Please resolve the errors and rerun..'
                } else
                    println(ANSI_BOLD + ANSI_GREEN + "Found environment variable named hub_org with value as: " + hub_org + ANSI_NORMAL)
            }

            cleanWs()            
            checkout scm
            commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            build_tag = sh(script: "echo " + params.github_release_tag.split('/')[-1] + "_" + commit_hash + "_" + env.BUILD_NUMBER, returnStdout: true).trim()
            echo "build_tag: " + build_tag

            stage('Build') {
                env.NODE_ENV = "build"
                print "Environment will be : ${env.NODE_ENV}"
                sh '''
                    mvn clean install -DskipTests
                    mvn play2:dist -pl analytics-api
                '''
            }
            stage('Package') {
                dir('sunbird-analytics-service-distribution') {
                sh """
                   cp ../analytics-api/target/analytics-api-2.0-dist.zip .
                   /opt/apache-maven-3.6.3/bin/mvn3.6 package -Pbuild-docker-image -Drelease-version=${build_tag}
                   """
                }
            }
            stage('Retagging'){
               sh """
                docker tag sunbird-analytics-service:${build_tag} ${hub_org}/sunbird-analytics-service:${build_tag}
                echo {\\"image_name\\" : \\"sunbird-analytics-service\\", \\"image_tag\\" : \\"${build_tag}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json
                """
            }
            stage('ArchiveArtifacts') {
                archiveArtifacts "metadata.json"
                currentBuild.description = "${build_tag}"
            }
        }
    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }
}
