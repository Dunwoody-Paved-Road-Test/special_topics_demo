pipeline {
    agent any

    stages {
        stage('copy to nginx') {
            steps {
                script {
                    def remote = [:]
                    remote.name = "vagrant"
                    remote.host = "192.168.56.21"
                    remote.allowAnyHosts = true
                    remote.failOnError = true

                    withCredentials([usernamePassword(credentialsId: 'vagrant', passwordVariable: 'password', usernameVariable: 'username')]) {
                    remote.user = username
                    remote.password = password
                    }

                    sshPut remote: remote, from: './', into: '/var/www/data/static-web'
                }
            }
        }
        // stage('Validate HTML') {
        //     steps{
        //         script{
        //             httpRequest httpMode: 'POST', requestBody: '{ [ { "contents": contents }, { "email": changelogContext.commits[i].authorEmailAddress }, ] }', responseHandle: 'NONE', url: 'url', wrapAsMultipart: false
        //         }
        //     }
        // }
        stage('Email Notifications'){
            steps{
                script{
                    def changelogContext = gitChangelog returnType: 'CONTEXT'
                    def timestamp = changelogContext.commits.commitTime
                    def emailList = changelogContext.commits.authorEmailAddress
                    def emailBody
                    def check1
                    def check2

                    // format the workspace for the url 
                    def workspace = "$WORKSPACE"
                    workspace = workspace.split('/')
                    workspace = workspace[-1]
                    // start looping through the change log context
                    for (int i = 0; i < timestamp.size(); i++) {
                        check2 = check1
                        def time = timestamp[i]
                        def userName = emailList[i].split('@')
                        userName = userName[0]
                        def hourMin = time.split(':')
                        check1 = hourMin[0] + hourMin[1]
                        // if the previous timestamp has the same hour and minute
                        // this is if Jenkins detects more than one new commit during a scan. For example, two users commit 
                        // somehting at the same time, or within one minute of each other
                        if (check1 == check2) {
                            emailBody = 'Your static html is now available at http://98.240.222.112:49160/static-web/' + workspace + '/' + userName
                            emailext body: emailBody, subject: 'Paved-Road Auto Notification', to: emailList[i]
                        }
                        // if the first iteration of the loop
                        if (i == 0) {
                            // html validator goes here
                            emailBody = 'Your static html is now available at http://98.240.222.112:49160/static-web/'+ workspace + '/' + userName
                            emailext body: emailBody, subject: 'Paved-Road Auto Notification', to: emailList[i]

                            // validate html
                            // def htmlFilesList = findFiles excludes: '', glob: userName + '/'
                            // def htmlFiles
                            // for (int x = 0; x < htmlFilesList.size(); x++) {
                            //     def contents = readFile htmlFilesList[x].toString()
                            //     htmlFiles = htmlFiles + '{'
                            // }
                            // //def json = '{"users": [ {"userEmail":' + emailList[i] + ', "results": [ {"filename":' + 
                            // def reqBody = """
                            //     {​​​​​​​​
                            //         "users": [
                            //             {​​​​​​​​
                            //                 "userEmail": "$emailList[i]",
                            //                 "results": [
                            //                     {​​​​​​​​
                            //                         "fileName": "hate.html",
                            //                         "results": "Error: Start tag seen without seeing a doctype first. Expected “<!DOCTYPE html>”.\nFrom line 1, column 1; to line 1, column 6\nError: Element “head” is missing a required instance of child element “title”.\nFrom line 1, column 7; to line 1, column 12\nWarning: Consider adding a “lang” attribute to the “html” start tag to declare the language of this document.\nFrom line 1, column 1; to line 1, column 6\nThere were errors.\n"
                            //                     }​​​​​​​​,
                            //                     {​​​​​​​​
                            //                         "fileName": "love.html",
                            //                         "results": "Error: Element “head” is missing a required instance of child element “title”.\nFrom line 1, column 22; to line 1, column 27\nWarning: Consider adding a “lang” attribute to the “html” start tag to declare the language of this document.\nFrom line 1, column 16; to line 1, column 21\nThere were errors.\n"
                            //                     }​​​​​​​​
                            //                 ]
                            //             }​​​​​​​​
                            //         ]
                            //     }​​​​​​​​
                            // """
                            // httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: reqBody, responseHandle: 'NONE', url: 'url', wrapAsMultipart: false
                            
                            //def contents = readFile '$userName/'
                        }
                        else {
                            break
                        }
                    }
                }
            }
        }
        stage('Write Changelog'){
            steps{
                script{
                    def changelogString = gitChangelog returnType: 'STRING',
                        template: """{{#commits}}{{messageTitle}},{{authorEmailAddress}},{{commitTime}}
                        {{/commits}}"""
                    writeFile file: 'ChangeLog.txt', text: changelogString
                }
            }
        }
    }
}