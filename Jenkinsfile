
def inputParams = [:]
inputParams.configuration = inputParams.get('configuration', 'Release')
inputParams.maxcpucount = inputParams.get('maxcpucount', ':1')
inputParams.verbosity = inputParams.get('verbosity', ':minimal')
inputParams.platform = inputParams.get('platform', 'Any CPU')
inputParams.warningThreshold = inputParams.get('warningThreshold', 1)
inputParams.testRunner = inputParams.get('testRunner', 'dotnet')
inputParams.testPlatform = inputParams.get('testPlatform', 'x64')
inputParams.testCaseFilter = inputParams.get('testCaseFilter', '')
inputParams.nugetPackOutputDir = inputParams.get('nugetPackOutputDir', 'nuget-pack-out')
inputParams.agentLabel = inputParams.get('agentLabel', 'windows2004')
inputParams.dockerImage = inputParams.get('dockerImage', 'artifactorydk.3shape.local/threeshapedocker-features/threeshape.dotnet.framework.sdk.vcpp:4.8-wsc2004-378d7c7')
inputParams.dockerArgs = inputParams.get('dockerArgs', '')
inputParams.cmakePath = 'C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\Common7\\IDE\\CommonExtensions\\Microsoft\\CMake\\CMake\\bin\\cmake.exe'

inputParams.pipelineTimeoutMinutes = inputParams.get('pipelineTimeoutMinutes', 30)
inputParams.publishToArtifactory = inputParams.get('publishToArtifactory', false)

    // pipelineHelper(sharedLibrary: this, parameters: inputParams) {

        pipeline {
            agent {
                docker {
                    label inputParams.agentLabel
                    image inputParams.dockerImage
                    args  inputParams.dockerArgs
                }
            }
            options {
                timeout(time: inputParams.pipelineTimeoutMinutes, unit: 'MINUTES')
                disableConcurrentBuilds()
            }
            post {
                always {
                    powershell 'git clean -fdx'
                    powershell 'git lfs prune'
                }
            }

            stages {
                stage('Init') {
                    steps {
                        echo "inputParams: ${inputParams}"
                        powershell 'gci env:'
                    }
                }

                stage('Build solution') {
                    steps {
                        powershell 'cinst anaconda3 -y --no-progress -s https://artifactorydk.3shape.local/api/nuget/chocolatey --params \'"/AddToPath /D:c:\\anaconda"\''
                        powershell "where.exe python"
                        powershell "where.exe python3"
                        // powershell "python setup.py bdist_wheel"
                        powershell ".\\build.bat --config Release --parallel --build_nuget --use_openmp --cmake_path \"${inputParams.cmakePath}\""
                    }
                }

                // stage('Run unit tests') {
                //     when {
                //         expression { return detectTestProject() }
                //     }
                //     steps {
                //         runUnitTests    configuration: inputParams.configuration,
                //                         testRunner: inputParams.testRunner,
                //                         testPlatform: inputParams.testPlatform,
                //                         testCaseFilter: inputParams.testCaseFilter,
                //                         verbosity: inputParams.verbosity
                //     }
                //     post {
                //         always {
                //             step([$class: 'MSTestPublisher'])
                //         }
                //     }
                // }

                // stage('Pack') {
                //     steps {
                //         powershell "dotnet pack -c:${inputParams.configuration} --no-build --no-restore --output " +
                //             "'${inputParams.nugetPackOutputDir}' -v${inputParams.verbosity}"

                //         script {
                //             dotnetUtils.checkNugetPackageFilenames()
                //         }
                //     }
                // }

                // stage('Publish to Artifactory') {
                //     when {
                //         anyOf {
                //             buildingTag()
                //             expression { inputParams.publishToArtifactory }
                //         }
                //     }
                //     steps {
                //         uploadNugetToArtifactory nugetPackOutputDir: inputParams.nugetPackOutputDir
                //     }
                //     post {
                //         success {
                //             publishBuildInfoArtifactory()
                //         }
                //     }
                // }
            }//stages
        }//pipeline
    // }//pipelineHelper
