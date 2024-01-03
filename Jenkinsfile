pipeline {
    agent any

    tools {
        maven "MVN_HOME"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "100.25.2.101:8081"
        NEXUS_REPOSITORY = "parallel"
        NEXUS_CREDENTIAL_ID = "nexus"
    }

    stages {
        stage("Parallel Stages") {
            parallel {
                stage("Clone Code") {
                    steps {
                        script {
                            git 'https://github.com/betawins/sabear_simplecutomerapp.git'
                        }
                    }
                }

                stage("Build with Maven") {
                    steps {
                        script {
                            sh 'mvn -Dmaven.test.failure.ignore clean package'
                        }
                    }
                }

                stage("Publish to Nexus") {
                    steps {
                        script {
                            pom = readMavenPom file: "pom.xml"
                            filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                            artifactPath = filesByGlob[0].path
                            artifactExists = fileExists artifactPath

                            if (artifactExists) {
                                echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                                nexusArtifactUploader(
                                    nexusVersion: NEXUS_VERSION,
                                    protocol: NEXUS_PROTOCOL,
                                    nexusUrl: NEXUS_URL,
                                    groupId: pom.groupId,
                                    version: pom.version,
                                    repository: NEXUS_REPOSITORY,
                                    credentialsId: NEXUS_CREDENTIAL_ID,
                                    artifacts: [
                                        [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging],
                                        [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: "pom.xml",
                                        type: "pom"]
                                    ]
                                )
                            } else {
                                error "*** File: ${artifactPath}, could not be found"
                            }
                        }
                    }
                }
            }
        }
    }
}
