// This shows a simple build wrapper example, using the Timestamper plugin.
// node('master') ==
//                 \/
//                  same as node
// Simple echo without 'node' context
stage name: "Nodeless Sample Stage"
echo 'Hello Pipeline World!!!'

stage(" =~=~= Collect Build Info =~=~= ") {
	
	// node nested inside stage
	node ('master') {

		timestamps {
			
			// what's the diff between this and git url... ?
			checkout scm

			//check node type
			def nodeOSVal = isUnix()
			if (nodeOSVal) {
			   echo "==> This build is running on a Unix-type node"
			} else {
			   echo "==> This build is running on a Windows-type node"
			}

			// not including include patterns, includes **
			stash name: "trial-sources"

			def file_name = sh(returnStdout: true, script: 'git log -1 --abbrev-commit --pretty=oneline --name-only | tail -n 1')
			
			env.changeset = file_name
			writeFile([file: 'changeset.txt', text: file_name])
			archive('changeset.txt')

			try {
				// Extract basic build details
				env.build_branch = env.BRANCH_NAME
				env.build_link   = env.JOB_URL
			} catch(error) {
				throw error
			} finally {
				echo "Build details were gathered."
			}
		}
	}
}

node('linux') {
	// Spit out timestamps in console log for each step in various stages
	timestamps {

		// Display basic details about commit and the build
		echo "Fresh build on branch, ${build_branch} was triggered by the latest commit -- "
		echo "Changeset generated from the latest commit includes:"
		echo "${changeset}"
		echo "This build can be accessed via, ${build_link}"

		// python plugin does not support this step ?
		// py command: "print ${BUILD_NUMBER}'"

		// Just some echoes to show the timestamps.
		stage(' =~=~= OS and Python Version =~=~= ') {

			echo "Running on host : "
			// Exclude excessive output from console logs
			sh ('#!/bin/sh -e\n' + "echo `hostname`")
			echo "OS Version : "
			sh "pg /etc/debian_version"

			// Try calling a function to check pkg info
			// Example for nested stages
			stage(" =~=~= Python Info!!! =~=~= ") {
			    pkginfo 'python'
			}
	
			stage(" =~=~= Collect Env vars... =~=~= ") {
			    // Collect and print all env variables
			    sh 'env > env_vars.txt'
			    def envdump = readFile('env_vars.txt')
			    echo "== START: Dump of enviroment variables =="
			    echo "${envdump}"
			    echo "== END: Dump of enviroment variables =="
			}

			stage(" =~=~= Welcome Jenkins =~=~= ") {
			     dir("trial-sources") { 
			         unstash "trial-sources"
			     }
			     
			     // what's in the contents?
			     sh "ls -la ${pwd()}"
			     
			     sh "ls -la ${pwd()}/trial-sources"
			     
			     // Howto execute external Python script ?
			     sh "python ${pwd()}/trial-sources/welcome.py Jenkins"

			    // Using shared python library
			    pyfuel = libraryResource "${pwd()}/trial-sources/resources/math.py"
			    sh pyfuel
			}
		}
	}
}

@NonCPS
def printEnv() {
	// TODO => Why isn't this loop printing ? needs further research...
	/* sh 'env > env_vars.txt'
	   readFile('env_vars.txt').split("\r?\n").each {
		println it
	}
	*/
}


def pkginfo(pkgname) {
 
   echo "Trying to find details of ${pkgname} package"
   sh("dpkg-query -s ${pkgname}")

}

