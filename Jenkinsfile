pipeline {
   agent { label 'google-slave-sg' }
   
   parameters {
        // The space ID that we will be working with. The default space is typically Spaces-1.
        string(defaultValue: 'Spaces-1', description: '', name: 'SpaceId', trim: true)
        // The Octopus project we will be deploying.
        string(defaultValue: 'RandomQuotes-Java', description: '', name: 'ProjectName', trim: true)
        // The environment we will be deploying to.
        string(defaultValue: 'dev-cj', description: '', name: 'EnvironmentName', trim: true)
        // The name of the Octopus instance in Jenkins that we will be working with. This is set in:
        // Manage Jenkins -> Configure System -> Octopus Deploy Plugin
        string(defaultValue: 'OctopusDeploy', description: '', name: 'ServerId', trim: true)
    }
    //    parameters {
    //     booleanParam defaultValue: false, description: '', name: 'CreateSiteName_Rapid7'
    //     choice(name: 'ACTION', choices: ['Dryrun', 'Apply', 'Destroy'])
    // }
   stages {
        stage('checkout') {
            steps {
                script {
                echo "Now Perform Checkout Step"
                //echo "Checkout branch (${branchname})"
                cleanWs()

                def checkoutVars = checkout([$class : 'GitSCM', branches: [[name: 'master']],
                          userRemoteConfigs: [[url : 'https://github.com/pisineen/RandomQuotes-Java.git',
                          ]]])
                          env.GIT_URL = checkoutVars.GIT_URL
                          env.GIT_COMMIT = checkoutVars.GIT_COMMIT
                          env.GIT_BRANCH = checkoutVars.GIT_BRANCH
                }

            }
        }
       stage ('Add tools') {
            steps {
                sh "echo \"OctoCLI: ${tool('octocli')}\""
            }
        }
        stage('build and SonarQube analysis') {
            steps {
                 script {
                // Update the Maven project version to match the current build
                sh(script: "mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}", returnStdout: true)
                // Package the code
                //sh(script: "mvn package", returnStdout: true)
                
                withSonarQubeEnv("sonarqube"){
                sh "mvn clean package sonar:sonar -Dsonar.projectKey=octopus -Dsonar.projectVersion=1.0.${BUILD_NUMBER}"
                }
                
                }
                     
            }
        }
        stage('deploy') {
            steps {                
                octopusPack additionalArgs: '', includePaths: "${env.WORKSPACE}/target/randomquotes.1.0.${BUILD_NUMBER}.jar", outputPath: "${env.WORKSPACE}", overwriteExisting: false, packageFormat: 'zip', packageId: 'randomquotes', packageVersion: "1.0.${BUILD_NUMBER}", sourcePath: '', toolId: 'octocli', verboseLogging: false
                octopusPushPackage additionalArgs: '', overwriteMode: 'FailIfExists', packagePaths: "${env.WORKSPACE}/target/randomquotes.1.0.${BUILD_NUMBER}.jar", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'octocli'
                /*
                    Note that the gitUrl param is passed manually from the environment variable populated when this Jenkinsfile is downloaded from Git.
                    This is from the Jenkins "Global Variable Reference" documentation:
                    SCM-specific variables such as GIT_COMMIT are not automatically defined as environment variables; rather you can use the return value of the checkout step.
                    This means if this pipeline checks out its own code, the checkout method is used to return the details of the commit. For example:
                    stage('Checkout') {
                        steps {
                            script {
                                def checkoutVars = checkout([$class: 'GitSCM', userRemoteConfigs: [[url: 'https://github.com/OctopusSamples/RandomQuotes-Java.git']]])
                                env.GIT_URL = checkoutVars.GIT_URL
                                env.GIT_COMMIT = checkoutVars.GIT_COMMIT
                                env.GIT_BRANCH = checkoutVars.GIT_BRANCH
                            }
                            octopusPushBuildInformation additionalArgs: '', commentParser: 'GitHub', overwriteMode: 'FailIfExists', packageId: 'randomquotes', packageVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'Default', verboseLogging: false, gitUrl: "${GIT_URL}", gitCommit: "${GIT_COMMIT}", gitBranch: "${GIT_BRANCH}"
                        }
                    }
                */
                octopusPushBuildInformation additionalArgs: '', commentParser: 'GitHub', overwriteMode: 'FailIfExists', packageId: 'randomquotes', packageVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'octocli', verboseLogging: false, gitUrl: "${GIT_URL}", gitCommit: "${GIT_COMMIT}", gitBranch: "${GIT_BRANCH}"
                octopusCreateRelease additionalArgs: '', cancelOnTimeout: false, channel: '', defaultPackageVersion: '', deployThisRelease: false, deploymentTimeout: '', environment: "${EnvironmentName}", jenkinsUrlLinkback: false, project: "${ProjectName}", releaseNotes: false, releaseNotesFile: '', releaseVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", tenant: '', tenantTag: '', toolId: 'octocli', verboseLogging: false, waitForDeployment: false
                octopusDeployRelease cancelOnTimeout: false, deploymentTimeout: '', environment: "${EnvironmentName}", project: "${ProjectName}", releaseVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", tenant: '', tenantTag: '', toolId: 'octocli', variables: '', verboseLogging: false, waitForDeployment: true
            }
        }
      
       }

   }
