pipeline {
    agent { label 'master'}

    environment {
        DOCKER_CREDENTIALS = credentials('harbor-registry-credentials')
        CF_CREDENTIALS = credentials('pcfcreds')
        DOTNET_CLI_HOME = "/tmp"
        loginServer = "harbor.dev.cloudsprint.io"
        imageName = 'fego-apps/net35image'
        imageWithTag = "$loginServer/$imageName:latest"
        PCF_API_ENDPOINT = "api.sys.dev.cloudsprint.io"
        PCF_ORG_NAME = "FEGO-demo"
        PCF_SPACE_NAME = "FEGO-demo"
    }

    stages {

        stage('Build') {
            steps {
                sh 'echo'
                sh 'echo "Initial directory: "'
                sh 'pwd'
                script {
                    // Find all files with name "Dockerfile"
                    def files = findFiles(glob: '**/Dockerfile')
                    echo "Dockerfile files - $files"

                    // Loop through all dockerfile paths and then build the dotnet project
                    for (i = 0; i < files.size(); i++) {
                        echo "Building application ${i+1} at path -> ./${files[i].path}"
                        // Get the directory of the dockerfile (removing the filename)
                        def dockerfileDirectory = new File("${files[i].path}").parent

                        def directoryToExecute
                        if (dockerfileDirectory != null) {
                            directoryToExecute = "$workspace/${dockerfileDirectory}"
                        } else {
                            directoryToExecute = "$workspace"
                        }

                        dir ("$directoryToExecute")
                        {
                            sh 'echo "...... Building the project $dockerfileDirectory ......"'
                            sh 'dotnet build *.csproj'
                            echo "Done building app ${i+1}."
                            sh 'echo'
                        }
                    }
                    echo 'Done building all apps.'
                }
            }
        }

        stage ('Docker Build') {
            steps {
                // prepare docker build context

                // Build and push image with Jenkins' docker-plugin
                script {
                    def files = findFiles(glob: '**/Dockerfile')
                    echo "Dockerfile files - $files"

                    // Loop through all dockerfile paths and then build and push the docker images to the registry
                    for (i = 0; i < files.size(); i++) {
                        echo "Dockerizing application ${i+1} at path -> ./${files[i].path}"
                        def dockerfileDirectory = new File("${files[i].path}").parent

                        def directoryToExecute
                        if (dockerfileDirectory != null) {
                            directoryToExecute = "$workspace/${dockerfileDirectory}"
                        } else {
                            directoryToExecute = "$workspace"
                        }

                        dir ("$directoryToExecute")
                        {
                            echo 'Start - Build and push image with Jenkins docker-plugin'
                            sh 'docker login -u $DOCKER_CREDENTIALS_USR -p $DOCKER_CREDENTIALS_PSW $loginServer'

                            // Build and push image with Jenkins' docker-plugin
                            script {
                                def image = docker.build imageWithTag
                                image.push()
                                echo "Done pushing image ${i+1}."
                            }

                        }
                    }
                    echo 'Done pushing all images.'
                }
            }
        }

        stage ('Dev_Deployment') {
            steps{
                script {

                    def files = findFiles(glob: '**/manifest.yml')
                    echo "Manifest files - $files"

                    // Loop through all manifest.yml paths and then push the application to cloud foundry
                    for (i = 0; i < files.size(); i++) {
                        echo "Pushing application ${i+1} at path -> ./${files[i].path}"
                        def manifestDirectory = new File("${files[i].path}").parent

                        def directoryToExecute
                        if (manifestDirectory != null) {
                            directoryToExecute = "$workspace/${manifestDirectory}"
                        } else {
                            directoryToExecute = "$workspace"
                        }

                        dir ("$directoryToExecute")
                        {
                            sh 'pwd'
                            sh 'which cf'
                            sh 'cf --version'
                            sh 'cf login -a $PCF_API_ENDPOINT -u $CF_CREDENTIALS_USR -p $CF_CREDENTIALS_PSW'
                            sh 'cf t -o $PCF_ORG_NAME -s $PCF_SPACE_NAME'
                            sh 'CF_DOCKER_PASSWORD=$DOCKER_CREDENTIALS_PSW cf push'
                            echo "Done pushing app ${i+1} to CF."
                            sh 'echo'
                        }
                    }
                    echo 'Done pushing all apps to CF.'
                }
            }
        }

    }
}
