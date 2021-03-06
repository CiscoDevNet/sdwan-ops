pipeline {
    agent {
        dockerfile {
            filename 'Dockerfile'
            args  '-v /etc/passwd:/etc/passwd'
        }
    }
    options {
      disableConcurrentBuilds()
      lock resource: 'jenkins_sdwan'
    }
    environment {
        VIRL_USERNAME = credentials('cpn-virl-username')
        VIRL_PASSWORD = credentials('cpn-virl-password')
        VIRL_HOST = credentials('cpn-virl-host')
        VIRL_SESSION = "jenkins_sdwan"
        VIPTELA_ORG = credentials('viptela-org')
        LICENSE_TOKEN = credentials('license-token')
        HOME = "${WORKSPACE}"
        DEFAULT_LOCAL_TMP = "${WORKSPACE}/ansible"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [
                        [$class: 'SubmoduleOption', disableSubmodules: false, recursiveSubmodules: true]
                    ],
                    userRemoteConfigs: [[url: 'https://github.com/ciscodevnet/sdwan-devops']]
                ])
                checkout([$class: 'GitSCM',
                    branches: scm.branches,
                    doGenerateSubmoduleConfigurations: false,
                    extensions: scm.extensions + [
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: 'ops',],
                        [$class: 'SubmoduleOption', disableSubmodules: false, recursiveSubmodules: true]
                    ],
                    submoduleCfg: [],
                    userRemoteConfigs: scm.userRemoteConfigs
                ])
            }
        }
        stage('Build VIRL Topology') {
           steps {
                echo 'Running build.yml...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'build.yml'
                echo 'Configure licensing...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', extras: "-e license_token=${LICENSE_TOKEN}", playbook: 'configure-licensing.yml'
           }
        }
        stage('Configure SD-WAN Fabric') {
           steps {
                echo 'Configure SD-WAN Fabric'
                withCredentials([file(credentialsId: 'viptela-serial-file', variable: 'VIPTELA_SERIAL_FILE')]) {
                    ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', extras: '-e virl_tag=jenkins -e \'organization_name="${VIPTELA_ORG}"\' -e serial_number_file=${VIPTELA_SERIAL_FILE} -e viptela_cert_dir=${WORKSPACE}/myCA', playbook: 'configure.yml'
                }
                echo 'Waiting for vEdges to Sync...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'waitfor-sync.yml'                
                echo 'Loading Templates...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'import-templates.yml'
                echo 'Attaching Templates...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'attach-template.yml'
                echo 'Loading Policy...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'import-policy.yml'
                echo 'Activating Policy...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'activate-policy.yml'
           }
        }
        stage('Testing sdwan-ops') {
           steps {              
                echo 'Exporting Templates...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'ops/export-templates.yml'
/*
                echo 'Import Templates...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'ops/import-templates.yml'
*/
                echo 'Exporting Policy...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'ops/export-policy.yml'
/*
                echo 'Import Policy...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'ops/import-policy.yml'
*/
                echo 'Device Facts...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'ops/device-facts.yml'
           }
        }        
        stage('Running Tests') {
           steps {
                echo 'Check SD-WAN...'
                ansiblePlaybook disableHostKeyChecking: true, inventory: 'inventory/crn1', playbook: 'check-sdwan.yml'
           }
        }    
    }    
    post {
        always {
            ansiblePlaybook disableHostKeyChecking: true, playbook: 'clean.yml'
            cleanWs()
        }
    }          
}

