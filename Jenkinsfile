pipeline {
    agent any
    environment {
        HAB_NOCOLORING = false
        HAB_BLDR_URL = 'https://bldr.habitat.sh/'
        HAB_ORIGIN = 'nrycar'
    }
    stages {
        stage('Clone from GitHub') {
	    agent {
    	        node {
        	    label 'windows'
    		}
	    }
            steps {
                git url: 'https://github.com/ChefRycar/contosouniversity.git', branch: 'master'
            }
        }
        stage('Build Chef Habitat Artifact') {
	    agent {
    	        node {
        	    label 'windows'
    		}
	    }
            steps {
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                    script {
                       powershell 'hab origin key download $env:HAB_ORIGIN -z $env:HAB_AUTH_TOKEN --secret'
                       powershell 'hab origin key download $env:HAB_ORIGIN'
                       powershell 'echo hab pkg build . -D'
                    }
                    habitat task: 'build', directory: '.', origin: "${env.HAB_ORIGIN}", docker: false

                }
            }
        }
        stage('Upload to unstable channel on bldr.habitat.sh') {
	    agent {
    	        node {
        	    label 'windows'
    		}
	    }
            steps {
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                    habitat task: 'upload', authToken: env.HAB_AUTH_TOKEN, lastBuildFile: "${workspace}/results/last_build.env", bldrUrl: "${env.HAB_BLDR_URL}"
                }
            }
        }
        stage('Promote to Stable') {
	    agent {
    	        node {
        	    label 'windows'
    		}
	    }
            steps {
                script {
                    env.HAB_PKG = sh (
                        script: "curl -s https://bldr.habitat.sh/v1/depot/channels/nrycar/unstable/pkgs/contosouniversity/latest\\?target\\=x86_64-linux | jq '(.ident.name + \"/\" + .ident.version + \"/\" + .ident.release)'",
                        returnStdout: true
                        ).trim()
                }
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                  habitat task: 'promote', channel: "stable", authToken: "${env.HAB_AUTH_TOKEN}", artifact: "${env.HAB_ORIGIN}/${env.HAB_PKG}", bldrUrl: "${env.HAB_BLDR_URL}"
                }
            }
        }
    }
}
