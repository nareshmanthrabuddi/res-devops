#!groovy

/**
 *
 * Author:    Naresh Manthrabuddi
 * Created:   07.08.2019
 * 
 **/

pipeline {
  agent any
  
	stages {
	    stage('Pipeline gets Started') {		
			steps {
			   echo 'RES Project Pipeline Started'
			}
	    }
	}
}
node {

	def v_environment = ""
	def v_buildMechanism =""
	def v_muleRuntimeEnvironment =""
	def v_workers = ""
	def v_cores = ""
	def v_appExists= ""
	def v_applicationName = ""
	def v_anypointCredentialID = ""
	def v_anypointOrganization = ""
	def v_AnypointEnvironment = ""
	def v_muleEvn = ""
	def v_encryptKey = ""
	def v_artifactId = ""
	def v_version = ""
	def v_package = ""
	def v_downloadFilePath = ""

	properties([
     parameters([
       choiceParam(
         choices: 'DEV\nSIT\nUAT\nPROD',
            description: 'Please select the ENVIRONMENT for Deployment',
            name: 'ENVIRONMENT'
       ),
       choiceParam(
         choices: 'BUILD-MUNITS\nBUILD-MUNITS-SONAR\nBUILD-MUNITS-SONAR-RELEASE',
            description: 'Please select the Build Mechanism',
            name: 'BUILD_MECHANISM'
       ),
       choiceParam(
         choices: '4.1.3\n4.1.4\n4.1.5',
            description: 'Please select mule runtime version for Deployment?',
            name: 'MULE_RUNTIME_VERSION'
       ),
       choiceParam(
         choices: '1\n2\n3\n4\n5\n6\n7\n8',
            description: 'Please select the workerSize for Deployment?',
            name: 'WORKERS'
       ),
       choiceParam(
         choices: '0.1\n0.2\n1\n2\n4\n8\n16',
            description: 'Please select the vCores for Deployment?',
            name: 'VCORES'
       ),
       choiceParam(
         choices: 'YES\nNO',
            description: 'Is application existed on cloud?/Are you re-deploying application on Cloud?',
            name: 'APP_EXISTS'
       ),
       string(
		   name: 'APPLICATION_NAME', 
		   defaultValue: 'res-devops', 
		   description: 'Please enter the application name for CloudHub Deployment?'
	   ),
       choiceParam(
         choices: 'DEV_CREDENTIAL_ID\nSIT_CREDENTIAL_ID\nPROD_CREDENTIAL_ID',
            description: 'Please select the vCores for Deployment?',
            name: 'ANYPOINT_CREDENTIAL_ID'
       ),
       string(
		   name: 'ANYPOINT_ORGANIZATION', 
		   defaultValue: 'LnD', 
		   description: 'Please provide CloudHub Anypoint Organization name to deploy?'
	   ),
	   choiceParam(
         choices: 'DEV\nSIT\nUAT\nPROD',
            description: 'Please select the Cloudhub Anypoint Environment to deploy?',
            name: 'ANYPOINT_ENVIRONMENT'
       )
     ])
    ])

	if(params.BUILD_MECHANISM == 'BUILD-MUNITS') {
		try{
			stage 'Code Build & MUnits Execution'	
				UDF_BuildSourceCode()
					
			stage 'Notification'
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","success")
				
		} catch(error) {
			throw(error)
			SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")

		}		
	} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR') {
		try{
			stage 'Code Build & MUnits Execution'	
				UDF_BuildSourceCode()
				
			stage 'SonarQube Analysis'
				UDF_ExecuteSonarQubeRules()
					
			stage 'Notification'
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","success")

		} catch(error) {
			throw(error)
			SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
		}
	} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR-RELEASE') {
		try{
			stage 'Code Build & MUnits Execution'	
				UDF_BuildSourceCode()
				
			stage 'SonarQube Analysis'
				UDF_ExecuteSonarQubeRules()

			stage 'DeployToCloudHub'
				UDF_DeployToCloudHub()

			stage 'Notification'
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")	
		} catch(error) {
			throw(error)
			SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
		}
	}
}

/*
BUILD - STAGE
This function provides the functionality to build your clode
*/
def UDF_BuildSourceCode()
{	
	try	{
		echo 'Build is Starting'
		bat 'mvn -U install -DskipTests=true'	
		echo 'Build Completed'		
	}catch(error) {
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
SONARQUBE - STAGE
This function provides functionality to do the SONAR Analysis
*/
def UDF_ExecuteSonarQubeRules()
{	
	try{
		echo 'SonarQube Rules Execution started'
		//bat 'mvn sonar:sonar'
		echo 'SonarQube Rules Execution Completed'	
	} catch(error) {
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}
   
/*
DEPLOY STAGE
This function provides functionality to deploy the application package(zip file) to CloudHub Runtime
*/

def UDF_DeployToCloudHub() {	

	echo "###### Application Deployment Stage ######"

	v_environment = "${params.ENVIRONMENT}"
	v_buildMechanism ="${params.BUILD_MECHANISM}"
	v_muleRuntimeEnvironment ="${params.MULE_RUNTIME_VERSION}"
	v_workers = "${params.WORKERS}"
	v_cores = "${params.VCORES}"
	v_appExists= "${params.APP_EXISTS}"
	v_applicationName = "${params.APPLICATION_NAME}"
	v_anypointCredentialID = "${params.ANYPOINT_CREDENTIAL_ID}"
	v_anypointOrganization = "${params.ANYPOINT_ORGANIZATION}"
	v_anypointEnvironment = "${params.ANYPOINT_ENVIRONMENT}"
	v_muleEvn = ""
	v_encryptKey = ""
	v_artifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
	v_version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
	v_package = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
	v_downloadFilePath = "${env.WORKSPACE}\\target\\${v_artifactId}-${v_version}-${v_package}.jar"	

	echo "ENVIRONMENT is : ${v_environment}"
	echo "BUILD_MECHANISM is : ${v_buildMechanism}"
	echo "MULE_RUNTIME_VERSION is : ${v_muleRuntimeEnvironment}"
	echo "WORKERS : ${v_workers}"
	echo "VCORES : ${v_cores}"
	echo "APP_EXISTS is : ${v_appExists}"
	echo "APPLICATION_NAME is : ${v_applicationName}"
	echo "ANYPOINT_CREDENTIAL_ID is : ${v_anypointCredentialID}"
	echo "ANYPOINT_ORGANIZATION is : ${v_anypointOrganization}"
	echo "ANYPOINT_ENVIRONMENT is : ${v_anypointEnvironment}"	
	echo "Environment Workspace is : ${env.WORKSPACE}"	
	echo "Download File Path is : ${v_downloadFilePath}"
	
	if("${params.ANYPOINT_CREDENTIAL_ID}" == 'DEV_CREDENTIAL_ID') {

		v_anypointCredentialID = 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ANYPOINT_CREDENTIAL_ID}" == 'SIT_CREDENTIAL_ID') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ANYPOINT_CREDENTIAL_ID}" == 'PROD_CREDENTIAL_ID') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'
	}

	echo "Anypoint credential ID is : ${v_anypointCredentialID}" 

	withCredentials([usernamePassword(credentialsId: "${v_anypointCredentialID}",passwordVariable: 'ANYPOINT_PASSWORD',usernameVariable: 'ANYPOINT_USERNAME')]) {
		bat "mvn deploy -DmuleDeploy -Danypoint.username=${ANYPOINT_USERNAME} -Danypoint.password=${ANYPOINT_PASSWORD} -Denvironment=${v_anypointEnvironment} -DbusinessGroup=${v_anypointOrganization} -DapplicationName=${v_applicationName}"
	}
}

/*
This function returns the repository details
*/
def UDF_GetGitRepoName(){	
	try{
	def tokens = "${env.JOB_NAME}".tokenize('/')
    String repo = tokens[tokens.size()-2]
	
    //branch = tokens[tokens.size()-1]	
    //org = tokens[tokens.size()-3]	
	
	return repo
	}catch(error)
	{
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
This function returns the POM Data
*/
def UDF_GetPOMData(udfp_PomName, udfp_PropertyName){	
	try{
	def resultVal = ""
	def pomFile = readFile(udfp_PomName)
	def pom = new XmlParser().parseText(pomFile)
	def gavMap = [:]
	resultVal =  pom[udfp_PropertyName].text().trim()
	return resultVal
	}catch(error)
	{
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
This function Sends Email
*/
def SendEmail(udfp_ToAddress, udfp_FromAddress, udfp_Status)
{
	try{
	   String body = ""
	   
	   if(udfp_Status == "success")
	   {
		   body= "SUCCESS"
	   }
	   else
	   {
		   body= "FAILED"
	   }
		/*
		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: "naresh.manthrabuddi@whishworks.com",
			 replyTo: "naresh.manthrabuddi@whishworks.com",
				from: "naresh.manthrabuddi@whishworks.com"
		*/
				
	}catch(error)
	{		
		throw(error)
	}
}
