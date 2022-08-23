#### 1. Variables

- Shell variable
  
  - Shell variables are variables that are limited in scope to the shell in which they were created.
  
  - `set`: list all shell variables. Example: `set | head -4`
  
  - Defining shell variables:
    
    - `var_name`=value    # No spaces around "="
    
    - `unset` `var_name`    # deletes var_name

- Environment variable
  
  - Environmental variables are just like shell variables, except they have extended scope: They persist in any child processes spawned by the shell in which they originate.
  
  - You can extend any shell variable to an environment variable  by applying the ‘export’ command to it.
    
    - example: `export var_name`
  
  - `env`: list all environment variables

#### 2. Metacharacters

- `#`: precedes a comment. *e.g.* `# This is a comment`

- `;`: command separator. *e.g.* `pwd;date`

- `*`: filename expansion wildcard. *e.g.* `ls /bin/ba*`

- `?`: single character wildcard in filename expansion. *e.g.* `ls /bin/?ash`

- `\`: escape special character interpretation. *e.g.* `echo "\$1 each"` # out: $1 each

- `""`: interpret literally, but evaluate metacharacters. *e.g.* `echo "$1 each"` # out: each

- `''`: interpret literally. *e.g.* `echo '$1 each'` #out: $1 each

#### 3. I/O redirection

- `>`: Redirect output to file

- `>>`: Append output to file

- `2>`: Redirect standard error to file

- `2>>`: Append standard error to file

- `<`: Redirect file contents to standard input

#### 4. Command substitution

- Replace command with its output
  
  `$(command)` or ``command` `

- *e.g.* Store output of `pwd` command in 'here':
  
  `here=$(pwd)` 
  
  `echo $here` #out: /home/toannv

#### 5. Batch vs. concurrent modes

- Batch mode:
  
  - Commands run sequentially
    
    `command1; command2`

- Concurrent mode:
  
  - Commands run in parallel
    
    `command1 & command2`

#### 6. Scheduling Jobs using Cron

- Job scheduling
  
  - Schedule jobs to run automatically at certain times
  
  - Cron allows you to automate such tasks

- What are cron, crond, and crontab?
  
  - Cron is a service that runs jobs
  
  - Crond interprets 'crontab files' and submits jobs to cron
  
  - A crontab is a table of jobs and schedule data
  
  - Crontab command invokes text editor to edit a crontab file

- Scheduling cron jobs with rontab
  
  - `crontab -e    # open editor`
  
  - Job syntax:
    
    - m h dom mon dow command
  
  - Example job:
    
    - 30 15 * * 0 date >> sundays.txt
  
  - Viewing cron jobs:
    
    `crontab -l | tail -[number]`
    
    *e.g.* `crontab -l | tail -6` 
    
    
















