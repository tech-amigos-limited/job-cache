def jobBuildCacheUpload(bucketName, path, includes, excludes, jenkinsCredentialsId) {

    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${jenkinsCredentialsId}" , secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
        sh "aws s3 sync ${path} s3://${bucketName}/${JOB_NAME}/ --delete"
    }
}

def jobBuildCacheDownload(bucketName, path, includes, excludes, jenkinsCredentialsId) {

    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${jenkinsCredentialsId}" , secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
        
        sh "aws s3 sync s3://${bucketName}/${JOB_NAME}/ ${path} --delete"
    }
}

def jobBuildCacheUploadTar(bucketName, path, includes, excludes, jenkinsCredentialsId) {
    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${jenkinsCredentialsId}" , secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']])
    {
        def tarString = replace.JOB_NAME('/','_')

        sh """
        rm -f /tmp/${tarString}
        tar cf /tmp/${tarString} ${path}
        aws s3 cp /tmp/${tarString} s3://${bucketName}/
        """
    }
}

def jobBuildCacheDownloadTar(bucketName, path, includes, excludes, jenkinsCredentialsId) {
withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${jenkinsCredentialsId}" , secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']])
    {
        def tarString = replace.JOB_NAME('/','_')

        def checkCacheinS3 = sh (
            returnStatus: true,
            script: "aws s3 ls s3://${bucketName}/${tarString}"
        )

        if(checkCacheinS3 != "") {
            sh """
            rm -f /tmp/${tarString}
            aws s3 cp s3://${bucketName}/${tarString} /tmp/${tarString}
            tar xvf /tmp/${tarString} -C ${path}
            """
        }
    }
}

def purgeCache(bucketName, cacheName, jenkinsCredentialsId)
{
    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${jenkinsCredentialsId}" , secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']])
    {
        def checkCacheinS3 = sh (
                returnStatus: true,
                script: "aws s3 ls s3://${bucketName}/${cacheName}"
            )

            if(checkCacheinS3 != "") {
                sh """
                aws s3 rm s3://${bucketName}/${cacheName}
                """
            }
    }
}

pipeline {
    agent any

    stages {
        stage('Build Cache') {
            steps {
                script {
                    def bucketName = 'jenkins-jobs-cache'
                    def includes   = "cache_folder/**"
                    def excludes   = "*"
                    def path       = "${WORKSPACE}/cache_folder/"
                    def jenkinsCredentialsId = 'techamigo-creds'
                    def jobName = "${JOB_NAME}"

                    // jobBuildCacheDownload(bucketName,path,includes,excludes,jenkinsCredentialsId)

                    // purgeCache(bucketName,jobName,jenkinsCredentialsId)
                    
                    jobBuildCacheDownloadTar(bucketName,path,includes,excludes,jenkinsCredentialsId)

                    sh """
                        touch ${WORKSPACE}/cache_folder/test4.txt
                        rm -f ${WORKSPACE}/cache_folder/test3.txt
                    """
                    jobBuildCacheUploadTar(bucketName,path,includes,excludes,jenkinsCredentialsId)

                    // jobBuildCacheUpload(bucketName,path,includes,excludes,jenkinsCredentialsId)
                
                }
            
            }
        }
    }
}