pipeline {
agent any

stages {
stage ('Create Build Parameters'){
    steps{
        sh 'echo !---SETUP---!'
        git credentialsId: '', url: 'https://github.com/punitGour/JenkisDevOps.git'
        httpRequest acceptType: 'APPLICATION_JSON', authentication: '', consoleLogResponseBody: false, contentType: 'APPLICATION_JSON', outputFile: 'Repos.json', responseHandle: 'NONE', url: 'https://api.github.com/search/code?q=org:punitGour+filename:cloud.deployment&per_page=100'
        readFile 'Repos.json'
        stash includes: '**', name: 'build'
        script {
            node{
                unstash 'build'
                env.WORKSPACE = pwd()
                def buildConfig = load "GenerateBuildSelections.Groovy"
                def repos = buildConfig.GetListOfRepos("${env.WORKSPACE}/Repos.json")

                def dataMap = new HashMap<String,List>()
                for(i = 0; i < repos.size(); i++){
                    def repoName = repos[i]
                    httpRequest acceptType: 'APPLICATION_JSON', authentication: '', consoleLogResponseBody: false, contentType: 'APPLICATION_JSON', outputFile: "branches_${repoName}.json", responseHandle: 'NONE', url: "https://api.github.com/repos/punitGour/${repoName}/branches"
                    env.WORKSPACE = pwd()
                    def branches = buildConfig.GetListOfBranches("${env.WORKSPACE}/branches_${repoName}.json")
                    dataMap.put("${repoName}", "${branches}")

                }
                def builder = new groovy.json.JsonBuilder(dataMap)
                new File("/home/service/BuildSelectionOptions/options.json").write(builder.toPrettyString())
            }
        }
    }
}
}
post {
always {
    sh 'echo !---Cleanup---!'
    cleanWs()
}
}
}
