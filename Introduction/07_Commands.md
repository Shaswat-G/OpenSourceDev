# Linux Commands
This file is meant as a reference for non-basic commands in Linux. 

## About Commands
A command is either a bulit-in shell utility, an executable binary or script, alias or keyword.
1. type - tells you what the command is, a keyword, alias, a shell-built in or an executable.
2. which - (TODO)
3. whatis - (TODO)
4. whereis - (TODO)

## Environment Variables
Key-value pairs (names) that a terminal session has access to. They can be created, inspected, set and deleted. shell vairables only exist in the current session and can only be accesses by inside the terminal and not by any child processes, and also lost once the session closes. Env vars exported during a session can be accessed by child processes but do not persist across sessions. For that, we can add an export line to the ~/.zshrc file.

1. env - lists all the current environemnt vairables in the session. It has USER, PATH, PWD, SSH_AGENT_PID, SHELL and many more
2. export NEW_ENV_VAR="Hi" : the export command creates and sets a new env variable, visible in env
3. export $PATH="$PATH:$HOME/my_script.sh" - this. modifies the current path env variable to include your script. make sure to give it execute permisisons.
4. unseet NEW_ENV_VAR : to unset the env var.

## Redirects and Streams:
Linux distros and modern descendants are multi-user and multi-process where each process (say when you execute a command on the termial) has three streams, which contain data in *plain text*:
1. stdin (ID 0): Default keyboard
2. stdout (ID 1): Outcome of the command, streams to the terminal display. Can be redirected using ">" or ">>" which appends insteaad of overwriting.
3. stderr (ID 2) : Error mwssages, also streams to the terminal display.

Pipes "|" can be used to redirect the stdout of one command to the stdin of another to compose commands in a sequence.
Eg: ls -l | wc -l

stdin can be redirected from a file using "<".

Eg: wc -l < file.txt

To redirect stderr, use 2> or 2>> to specicy that we are talking about id 2.
To log both stderr and stdout in the same way, use 2>&1 (goes in append mode already, nothing line 2>>&1)

To discard error messaegs, you can redirect stderror to /dev/null

command < input-file : reads from input-file
command > output-file : create/overwrites output-file
command >> output-file :  Append to output-file

## Archiving and Compressing:
Many times we need to archive a directory tree (with multiple subdirectories and files) into one tape archive, we use tar for this. We optionally run compression on it using gzip to make the size less for transmisison. I personally have used scp for transferring a .tar.gz (gzip compressed taped archive) from a remote ssh server to my local machine and vice versa.

Tar has a greater size than the sum of the file sizes since it also store metadata.
1. tar -czvf new_archive.tar.gz directore/path/
2. tar -tzvf new_archive.tar.gz to view the contents
3. tar -xzvf new_archite.tar.gz -C new_directory/

## Symlinks
Creation of a file creates a unique INode number that identifies a file, its data container and metadata that the kernel can refer to a file as. A symlink is a soft link that points to the same INode.

learngitbranching.js.org?