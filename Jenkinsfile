pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('Transform') {
            agent {
                docker {
                    image 'cloudfluff/databaker'
                    reuseNode true
                }
            }
            steps {
                sh 'jupyter-nbconvert --to python --stdout sic2007.ipynb | python'
            }
        }
        stage('Publish results') {
            steps {
                script {
                    configFileProvider([configFile(fileId: 'pmd', variable: 'configfile')]) {
                        def config = readJSON(text: readFile(file: configfile))
                        String PMD = config['pmd_api']
                        String credentials = config['credentials']
                        def drafts = drafter.listDraftsets(PMD, credentials, 'owned')
                        def jobDraft = drafts.find { it['display-name'] == env.JOB_NAME }
                        if (jobDraft) {
                            drafter.deleteDraftset(PMD, credentials, jobDraft.id)
                        }
                        def newJobDraft = drafter.createDraftset(PMD, credentials, env.JOB_NAME)
                        String graph = "http://www.ons.gov.uk/dev/sic2007"
                        drafter.deleteGraph(PMD, credentials, newJobDraft.id, graph)
                        drafter.addData(PMD, credentials, newJobDraft.id,
                                        readFile(file: "sic2007.ttl", encoding: 'UTF-8'),
                                        'text/turtle;charset=UTF-8', graph)
                        drafter.publishDraftset(PMD, credentials, newJobDraft.id)
                    }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts '*.ttl'
        }
    }
}
