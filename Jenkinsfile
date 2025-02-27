
def output = ""
def sendEmail(jobName, jobId, jobStatus, output)
{
    echo "Job ${jobName} with ${jobId} is ${jobStatus}"
    echo output
}
pipeline {
    agent any
    parameters {
        string ( defaultValue: 'default',
        description: 'description',
            name: "ARTIFACT_NAME"
            )
        booleanParam defaultValue: false, name: 'FAIL_PIPELINE'
        booleanParam defaultValue: true, name: 'RUN_TEST'
    }
   
    stages{
        stage("Download"){
            steps{
                cleanWs()
                echo (message: "some string")
                dir('dir'){
                    git(
                        branch: "pipeline",
                        url: "https://github.com/KLevon/jenkins-course"
                        )
                    }
                rtDownload(
                    serverId: "Artifactory",
                    spec: '''{
                        "files": [
                        {
                            "pattern": "generic-local/libraries/printer.zip",
                            "target": "./"
                        }
                        ]
                        
                    }'''
                    )
                unzip (
                    zipFile: "libraries/printer.zip",
                    dir: "dir/"
                    )
            }
            
        }
        stage("Build"){
        steps{
            dir('dir'){
                bat(
                    script: """
                        Makefile.bat
                        """
                    )
                }
            
           // withCredentials(
             //   [usernamePassword(credentialsId: 'CRED_ID', passwordVariable: "psw", usernameVariable: "usr")
              //      ])
              //      {
               //         echo psw
                //        echo usr
                 //   }
            }
            
        }
        stage("Dynamic"){
            when { branch "feature/multi/*" }
            steps{
                echo "This is Dynamic stage!"
                }
            }
        stage("Tests"){
            when {
                equals expected: true, 
                actual: params.RUN_TEST
            }
            steps{
                script{
                    def array = ["printer", "scanner", "main"]
                    for (element in array) {
                       output += bat(
                            script: """
                                cd dir
                                Tests.bat $element
                                """, returnStdout: true
                            )
                            } 
                    }
                
                }
        }
        stage("Publish"){
        steps{
            zip(
                zipFile: "pipeline.zip",
                archive: true,
                dir: "dir/"
                )
            
            rtUpload(
                    serverId: "Artifactory",
                    spec: """{
                        "files": [
                        {
                            "pattern": "pipeline.zip",
                            "target": "generic-local/release/petar/${env.BUILD_ID}/${params.ARTIFACT_NAME}.zip"
                        }
                        ]
                        
                    }"""
                    )
            
            script{
                    if (params.FAIL_PIPELINE == true)
                    {
                        bat(
                                script: """
                                    exit 1
                                    """
                            )
                    }
                }
            }
            
            
        
        }
    
    }
    post{
        failure{
            sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
        }
        success{
            sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
        }
    }
}
