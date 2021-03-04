
def inputParams = [:]
inputParams.configuration = 'Release'
inputParams.maxcpucount = ':1'
inputParams.verbosity = ':minimal'
inputParams.platform = 'Any CPU'
inputParams.warningThreshold = 1
inputParams.testRunner = 'dotnet'
inputParams.testPlatform = 'x64'
inputParams.testCaseFilter = ''
inputParams.nugetPackOutputDir = 'nuget-pack-out'
inputParams.agentLabel = 'windows2004'
inputParams.dockerImage = 'artifactorydk.3shape.local/threeshapedocker/threeshape.dotnet.framework.sdk.vcpp:4.8-wsc2004'
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
                powershell returnStatus: true, script: 'where.exe python'
                powershell returnStatus: true, script: 'where.exe nuget'
                powershell 'pip install flatbuffers'
            }
        }

        stage('Build') {
            steps {
                powershell ".\\build.bat --config Release --parallel --build_nuget --use_openmp --cmake_path \"${inputParams.cmakePath}\" --ctest_path \"${inputParams.ctestPath}\" --cmake_generator \"Visual Studio 16 2019\""
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
