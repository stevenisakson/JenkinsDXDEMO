#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME
    def DEV_USERNAME = env.DEV_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def SANDBOX_CONSUMER_KEY = env.SANDBOX_CONSUMER_KEY
    def SFDC_HOST_SANDBOX = env.SFDC_HOST_SANDBOX_DH
    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Update CLI') {
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" update"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" update"
              }
            if (rc != 0) {
                error 'upgrade failed'
            }
            
          }
        stage('Create Scratch Org') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
              if (isUnix()) {
                rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/enterprise-scratch-def.json --json --setdefaultusername"
              }else{
                   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
              }
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
            def beginIndex = rmsg.indexOf('{')
            def endIndex = rmsg.indexOf('}')
            println(beginIndex)
            println(endIndex)
            def jsobSubstring = rmsg.substring(beginIndex)
            println(jsobSubstring)
            
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(jsobSubstring)
            //if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null
            
        }
        
          stage('Push To Test Org') {
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }
            if (rc != 0) {
                error 'push failed'
            }
            
          }
          stage('Open Scratch Org'){
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:org:open --targetusername ${SFDC_USERNAME}"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:open --targetusername ${SFDC_USERNAME}"
              }
            if (rc != 0) {
                error 'open failed'
            }
          }
          stage('Deploy to Sandbox'){
              if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant -i ${SANDBOX_CONSUMER_KEY} -u ${DEV_USERNAME} -f ${jwt_key_file} -r ${SFDC_HOST_SANDBOX}"
                }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant -i ${SANDBOX_CONSUMER_KEY} -u ${DEV_USERNAME} -f \"${jwt_key_file}\" -r ${SFDC_HOST_SANDBOX}"
                }
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:deploy -u ${DEV_USERNAME} -p force-app"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:deploy -u ${DEV_USERNAME} -p force-app"
              }
              
              if (rc != 0){
                  error 'deployment failed'
              }
          }
             
    }
}
