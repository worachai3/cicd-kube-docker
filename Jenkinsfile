pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment{
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.22.111'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        registryCredential = 'ecr:ap-southeast-1:awscreds'
        // appRegistry = '568406210619.dkr.ecr.ap-southeast-1.amazonaws.com/vprocontainers/vprofileapp'
        // dbRegistry = '568406210619.dkr.ecr.ap-southeast-1.amazonaws.com/vprocontainers/vprofiledb'
        // rmqRegistry = '568406210619.dkr.ecr.ap-southeast-1.amazonaws.com/vprocontainers/vprofilermq'
        vprofileRegistry = 'https://568406210619.dkr.ecr.ap-southeast-1.amazonaws.com/'
    }

    stages{
        stage("Build"){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post{
                success{
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage("Test"){
            steps{
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstayle Analysis'){
            steps{
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis'){
            environment{
                scannerHome = tool "${SONARSCANNER}"
            }

            steps{
                withSonarQubeEnv("${SONARSERVER}"){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest \
                    -Dsonar.junit.reportPaths=target/surefire-reports/ \
                    -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("Quality Gate"){
            steps{
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image'){
            steps{
                script{
                    dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER", '.')
                }
            }
        }

        stage('Upload App Image'){
            steps{
                script{
                    docker.withRegistry(vprofileRegistry, registryCredential){
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }


        stage('Remove Unused Docker Images') {
            steps {
                sh "docker rmi $appRegistry:$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deploy'){
            agent {label 'KOPS'}
                steps {
                    sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts"
                }
        }
    }
}