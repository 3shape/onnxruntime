
def inputParams = [:]
inputParams.configuration = 'Release'
inputParams.maxcpucount = ':1'
inputParams.verbosity = ':minimal'
inputParams.platform = 'Any CPU'
inputParams.warningThreshold = 1
inputParams.testRunner = 'dotnet'
inputParams.testPlatform = 'x64'
inputParams.testCaseFilter = ''
inputParams.agentLabel = 'windows2004'
inputParams.dockerImage = 'artifactorydk.3shape.local/threeshapedocker/threeshape.dotnet.framework.sdk.vcpp.cuda:4.8-servercore-amd64'
inputParams.dockerArgs = ''
inputParams.cmakePath = 'C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\Common7\\IDE\\CommonExtensions\\Microsoft\\CMake\\CMake\\bin\\cmake.exe'
inputParams.ctestPath = 'C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\Common7\\IDE\\CommonExtensions\\Microsoft\\CMake\\CMake\\bin\\ctest.exe'

inputParams.pipelineTimeoutMinutes = 60
inputParams.publishToArtifactory = false

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
            powershell 'git submodule foreach --recursive git clean -ffdx'
            powershell 'git reset --hard --recurse-submodule'
            powershell 'git submodule update --init --recursive'
        }
    }

    stages {
        stage('Setup') {
            steps {
                echo "inputParams: ${inputParams}"
                powershell 'gci env:'
                powershell 'cinst miniconda3 -y --no-progress --params \'"/AddToPath:1"\''
                powershell 'pip install flatbuffers'
            }
        }

        stage('Build') {
            steps {
                dotnetTool tool: 'gitversion'
                powershell ".\\build.bat --config ${inputParams.configuration} --parallel --skip_tests --build_nuget --use_dml --use_openmp --cmake_path \"${inputParams.cmakePath}\" --ctest_path \"${inputParams.ctestPath}\" --cmake_generator \"Visual Studio 16 2019\""
            }
        }

        stage('Publish to Artifactory') {
            when {
                anyOf {
                    buildingTag()
                    expression { inputParams.publishToArtifactory }
                }
            }
            environment {
                managedOutputPath = "csharp/src/Microsoft.ML.OnnxRuntime/bin/${inputParams.configuration}"
                nativeOutputPath = "build/Windows/Release/${inputParams.configuration}"
            }
            steps {
                uploadNugetToArtifactory nugetPackOutputDir: managedOutputPath
                uploadNugetToArtifactory nugetPackOutputDir: nativeOutputPath
            }
            post {
                success {
                    publishBuildInfoArtifactory()
                }
            }
        }
    }//stages
}//pipeline
