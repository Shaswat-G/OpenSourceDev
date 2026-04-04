# Linux Commands
This file is meant as a reference for non-basic commands in Linux. 

## About Commands
A command is either a bulit-in shell utility, an executable binary or script, alias or keyword.
1. type - tells you what the command is, a keyword, alias, a shell-built in or an executable.
2. apropos - Fuzzy seach for commands over documentation, behaves like a google search. Use to generate candidate commands for a task.
2. whatis - Gives a quick one line description / definition for quick reference
3. command --help - gives you a shallow usage and options, program determined, not standardized.
4. which - gives you the path of the executable that will run the command.
5. whereis - gives you the path of all the executables that will run the command, useful when you have multiple versions of a command installed and want to know which one is being used. Searches over the executables and man pages.

## Shell
A shell is a command line interpreter that provides a UI for users to run commands. A login shell is a shell that requires a login. An interactive shell is a shell where input and output streams are connected to the terminal. A non-interactive shell is a shell where input and output streams are not connected to the terminal, such as when running a script. A shell can be both login and interactive, or just one of them.

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

## Textprocessing
We compose a variety of commands together to achieve powerful text-processing.

## Regex:
Regular expressions are a string matching language that is used to search and extract patterns in text. They are used in many commands like grep, sed, awk, find etc. They have syntax to specify literals, character calsses, quantifiers, anchors, groups and metacharacters.


### Grep:
Grep is used for searching "lines" in text (stdout of a command or file). Primarily used for filtering.
Useful flags : i (case insensitive search), n (line numbers), v (invert search), R (recursive search in directory), E (regex pattern). Can be used for searching errors, warnings, timeouts, fatal in server logs, todos in a directory, non-comment lines in a config, etc.
Eg: git -iR TODO .
git -E "error|warning|fatal" < server_logs.log

### fd
fd is a friendlier version of find, and it is used to find files and directories.
Useful flags: -e (extension), -t (f for only files, d for only directories), 
Eg: fd -e py code/repo/package/module
fd -e log /var/log/ | xargs wc -l


### cut
cut is used to extract a fields (a range of fields, a set of fields starting from 1) from a delimited text structure. f for field and c for character
eg: cut -d, -f1-5 data.csv
grep -niv "^#" /etc/passwd | cut -d: -f1,5

### paste
Opposite of cut. It combines text streams together with a delimiter on a line by line basis.
paste -d, names.txt scores.txt

### expand and unexpand 
used to standardize tabs and spaces.

### sort
sorts text. -n (for numberical), -t, k2 (delimiter , and field 2). sort names.txt | uniq
Uniq only removes adjacent duplicates.

### tr
Character translation. .lower(), .upper(), reduce spaces, trim spaces, etc.
echo "hello" | tr 'a-z' 'A-Z'
echo "he2l4l5o" | tr -d '0-9'
echo "hellllooo" | tr -s "l,o"
echo "a,b,c,d" | tr ',' '\n'

fd -e md -t f | xargs wc -w | sort -n | tr -s " " | tr " " "," | cut -d, -f2

### tee
allws you to display what is written to file on screen too. Helps debug pipelines.


Most frequently appearing words:
tr ' ' '\n' < text.txt | sort | uniq -c | sort -n

### awk
One of the most powerful extraction tools - lets you select, filter, aggregate, project text by treating it as line by line records with fields as delimiter separated columns.

$0 -> represents the entire record (line)
$1, $2, ... -> represent the first field, second field, and so on
NL -> Number of fields
NR -> Number of records


## Process Management
  kill

  Send a signal to a process, usually related to stopping the process.
  All signals except for SIGKILL and SIGSTOP can be intercepted by the process to perform a clean exit.
  More information: https://manned.org/kill.1posix.

  Terminate a program using the default SIGTERM (terminate) signal:

    kill process_id

  List available signal names (to be used without the SIG prefix):

    kill -l

  Terminate a program using the SIGHUP (hang up) signal. Many daemons will reload instead of terminating:

    kill -HUP process_id

  Terminate a program using the SIGINT (interrupt) signal. This is typically initiated by the user pressing <Ctrl c>:

    kill -INT process_id

  Signal the operating system to immediately terminate a program (which gets no chance to capture the signal):

    kill -KILL process_id

  Signal the operating system to pause a program until a SIGCONT ("continue") signal is received:

    kill -STOP process_id

  Send a SIGUSR1 signal to all processes with the given GID (group id):

    kill -SIGUSR1 -group_id


To run a Linux command in the background, all you have to do is to add an ampersand (&) at the end of the command, like this:

your_command &

Background processes are those that run without user interaction, allowing the shell to be used for other commands simultaneously. These processes are non-blocking, meaning they do not prevent you from executing additional commands while they are running.

A background process is typically started by appending an & at the end of the command. For instance, when you run:

$ tar -czf archive.tar.gz large_directory/ &

Foreground processes are those that run directly in the terminal and require user interaction. These processes involve direct interaction with the user, receiving input and displaying output directly to the terminal.

When a foreground process runs, it blocks the shell, preventing other commands from being executed until the process completes.

For example, running a text editor like nano in the terminal will block further commands until the editor is closed.

Resuming a Background Process
If you have a paused process or one that's already running in the background, you can make it start running again using the bg (background) command. To resume the most recently paused or backgrounded job, simply type:

$ bg %1

To bring it back to the foreground, use fg %1. Similar to bg, you use the job number:

$ fg %1