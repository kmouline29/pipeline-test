pipeline {
    agent none
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '30'))
    }
    parameters {
        extendedChoice(
            name: 'TEST_OPTIONS',
            description: 'select which cloudrunner tests to run',
            type: 'PT_CHECKBOX',
            visibleItemCount: 9,
            multiSelectDelimiter: ',',
            defaultValue: 'txnid2,rejoins,resize,security,recover,export,dr,priorities,placemement-groups',
            value: 'txnid2,rejoins,resize,security,recover,export,dr,priorities,placemement-groups'
        )
        string(name: 'CLOUDRUNNER_OPTIONS', description:'options for cloudrunner')
        string(name: 'BRANCH', defaultValue: 'master', description: 'internal branch to use')
        string(name: 'PRO_BRANCH', defaultValue: 'master', description: 'pro branch to use')
        string(name: 'OPERATOR_BRANCH', defaultValue: 'master', description: 'operator branch to use')
        string(name: 'CLOUD', defaultValue: 'gke', description: 'which cloud to use')
    }
    stages {
        stage('Clone Workspace') {
            steps {
                scm {
                    cloneWorkspace('kit-system-test-nextrelease-build', 'Any')
                }
                // scm {
                //     cloneWorkspace(sourceWorkspaceName: 'kit-system-test-nextrelease-build',
                //                     selector: [$class: 'WorkspaceSelector',
                //                             criteria: 'LAST_COMPLETED_BUILD'])
                // }
            }
        }
        stage('Prepare Tests') {
            steps {
                script {
                    params.TEST_OPTIONS.split(',').each { option ->
                        stage("test ${option}") {
                            try {
                            echo "#!/bin/bash --login -x\ncd pro/tests/apptests\nVOLTCORE=/../../../internal CLOUDRUNNER_OPTIONS=$CLOUDRUNNER_OPTIONS\nBRANCH=$BRANCH\nPRO_BRANCH=$PRO_BRANCH\nOPERATOR_BRANCH=$OPERATOR_BRANCH\njenkins-scripts/run-k8s-${option}.sh $CLOUD"
                            assert "${option}" != "resize"
                            } catch (Throwable t) {
                                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                                    error "Test ${option} failed"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}