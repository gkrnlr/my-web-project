#!/bin/bash
# import multiple remote git repositories to local dir

set -e

# settings / change this to your config
projectsToClone="$@"
remoteHost=onestash.verizon.com
remotePort=7999
remoteUser=git
remoteDir="/scm/enmecl/"
repoExtension=".git"
noOfProjects=${#projectsToClone[@]}
localCodeDir="${HOME}/bulkcheckin/"
branchName = "$projectsToClone[0]"

if [${noOfProjects}==1 && "$projectsToClone[1]"=="All"]; then
	remoteRepos=$(ssh -l $remoteUser $remoteHost -p $remotePort "ls $remoteDir")
	echo "$remoteRepos"
else
	remoteRepos="$projectsToClone"
    echo "Projects to be Cloned ... $remoteRepos"
fi

echo "$localCodeDir"

# if no output from the remote ssh cmd, bail out
if [ -z "$remoteRepos" ]; then
    echo "No results from remote repo listing (via SSH)"
    exit
fi

# for each repo found remotely, check if it exists locally
# assumption: name repo = repo.git, to be saved to repo (w/o .git)
# if dir exists, skip, if not, clone the remote git repo into it
echo "Remote repos before clone $remoteRepos"
for gitRepo in $remoteRepos
do
  localRepoDir=$(echo ${localCodeDir}${gitRepo}|cut -d'.' -f1)
  if [ -d $localRepoDir ]; then 	
		echo -e "Directory $localRepoDir already exits, skipping ...\n"
	else
		cloneCmd="git clone -b $branchName ssh://$remoteUser@$remoteHost:$remotePort$remoteDir"
		cloneCmd=$cloneCmd"$gitRepo$repoExtension $localRepoDir"
		
		echo "$cloneCmd"
		cloneCmdRun=$($cloneCmd 2>&1)

		echo -e "Running: \n$ $cloneCmd"
		echo -e "${cloneCmdRun}\n\n"
	fi
done
