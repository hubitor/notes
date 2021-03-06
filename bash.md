`date` : print the date\
`cal` : show a calendar\
`!!` : previous command eg: `sudo !!` (run previous command as super user)\
`top` : display Linux tasks (view detailed list of processes running on this computer)\
`ps` : view all user's currently running jobs\
`kill -9 <PID>` : kill process with PID\
`mkdir -p path/multiple/folders` : will create multiple directories even if they don't exist\
`printenv` : view list of environment variables\
`variable_name=value` : create new environment variable\
`export variable_name=value` : create new environment variable and make it available to sub-processes (commands called from this shell)\
`unset variable_name` : remove environment variable\
`echo $variable_name` : view value of env variable\
`alias name="script"` : create new bash alias\
`unalias name` : remove alias\ 
`sort` : sort the lines of a file our input alphabetically
`chmod +xrw file_name` : add executable, readable, writeable permissions to a file\
`chmod 777 file_name` : same as above\
`chmod -rxw file_name` : remove all permissions from a file\
`chmod 000 file_name` : same as above\
`chmown owner file_name` : change owner of file\
`users` : view all users currently logged in
`cat etc/passwd` : list of all users
`cat etc/group` : list of all groups

### xargs
`input | xargs <command>` : will loop over multiple arguments in input and run a command on each.\

eg1: ls all files & folders in current and previous two directories:
```
printf ".\n..\n../.." | xargs ls
=> .
pwd
=> ..:
/Programming
=> ../..:
/Documents
```

eg2: send each argument to stdout:
```
echo "a\nb\nc" | xargs echo
=> a b c
```

eg3: cd into and pwd the currrent working directory and previous two directories:
```

export lv=".\n..\n../.."
printf $lv | xargs -I % sh -c 'cd %; pwd %'

=> /Users/josh/Documents/Programming
=> /Users/josh/Documents
=> /Users/josh

NOTES: 
# export the three arguments as alias "lv"
# "-I %" replace occurences of each argument with names read from input (stdin?)
# "sh" command interpreter (bash / shell)
# "-c" read commands from the command string operand '' instead of stdin. Execute the following command as interpreted by this program
# sh -c spawns a non-login, non-interactive session of sh
# 'cd %; pwd %' command string operand we defined (must use single-quotes)
```

### Bash scripts
Bash script files are *.sh\
`echo "script" > filename.sh` : creates bash script\
`source filename.sh` : runs bash script\
`chmod +x script_name.sh` : create executable script\
`./sript_file` : execute bash script\
`\` : escape character (can be used as line break)

bash loops: http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html

#### Examples:
__loop and print 3 times__\
`"for i in {1..3}; do echo Welcome \$i times; done"` \
__quotation marks and '$' must be escaped if you want to use them:__\
`"for i in {1..3}; do echo \"Welcome \$i times\"; done"` 

```
declare -a names=("josh" "kate" "aaron")    

for i in "${names[@]}"
do
   echo "$i"
done

> josh
> kate
> aaron
```

__once again to echo this command into a bash file you'll have to escape all the special characters:__
```
declare -a names=(\"josh\" \"kate\" \"aaron\" \"noah\")
for i in \"\${names[@]}\"
do 
  echo \"\$i\"`
done
```

### Redirectin Input and Output

`"|"` : the pipe operator redirects the output (stdout) of the first stream to the input (stdin) stream of the second command\
`">"` : redirects the output of the first stream to a different location\
`"<"` : gets input from particular location (rather than the default stdin) __note__ this causes the data to 'flow' right to left on the command line, eg: `sort <(echo "1\n3\n2")`\
`"string" >&0` : redirect "string" to stdin\
`"string" >&1` : redirect "string" to stdout\
`"string" >&2` : redirect "string" to sterror

### Grep

`grep / grep -e` : Search / Search using a (Regexp pattern)\
`grep -r` : Search recursively\
`grep "a" ~/.bash_profile` : Search .bash_profile for lines with the pattern [/a/]\
`grep "js" -r ~/Documents/Programming` : Search the Programming directory file recursively for [/js/]

### Cron jobs
>Scheduling Tasks with cron jobs: https://code.tutsplus.com/tutorials/scheduling-tasks-with-cron-jobs--net-8800

`crontab -e` : Open list of cron jobs in an editor\
`* * * * * date > ~/date_time.txt` : append the date to the file date_time.txt every minute\

> cron jobs can be set up to run at particular minutes of each hour (0-59), particular hours of each day (0-23), particular days of each month (1-31), particular months of each year (1-12), or particular days of each week (0-6, Sun-Sat). This is what the five stars at the beginning of the command above represent, respectively. Replace them with specific numbers to run them on particular days or at particular times.

> If a job is to be run irrespective of, for instance, the day of the week, then the position that represents the day of the week (the 5th position) should contain a star (*). This is why the command above runs every minute (the smallest interval available). cron jobs can be set up to run only when the system is rebooted, with @reboot replacing the stars/numbers. Jobs can also be run a specific number of times per hour or day or at multiple specific times per hour / day / week / month / etc.
