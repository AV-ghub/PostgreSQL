[Ускоряем PostgreSQL с помощью Tuned](https://habr.com/ru/companies/otus/articles/859270/)  

<details><summary><h3><a href="https://manpages.ubuntu.com/manpages/focal/man1/pg_top.1.html">pg_top</a></h3></summary>

  #### NAME
         pg_top - display and update information about the top cpu PostgreSQL processes
  
  #### SYNOPSIS
         pg_top [ OPTIONS ] [ NUMBER ]
  
  #### DESCRIPTION
         pg_top displays the top processes on the system and periodically updates this information.
         Raw cpu percentage is used to rank the processes.  If number is given, then the top number
         processes will be displayed instead of the default.
  
         pg_top  makes a distinction between terminals that support advanced capabilities and those
         that do not.  This distinction affects the choice of defaults for certain options.  In the
         remainder  of  this  document,  an  "intelligent"  terminal  is  one  that supports cursor
         addressing, clear screen, and clear to end of line.  Conversely, a "dumb" terminal is  one
         that  does not support such features.  If the output of pg_top is redirected to a file, it
         acts as if it were being run on a dumb terminal.
  
  #### OPTIONS
         -C, --color-mode
                Turn off the use of color in the display.
  
         -I, --hide-idle
                Do not display idle processes.  By default, pg_top displays both  active  and  idle
                processes.
  
         -T, --show-tags
                List  all  available  color  tags  and  the  current  set  of  tests used for color
                highlighting, then exit.
  
         -W, --password
                Forces pg_top to prompt for a password before connecting to a database.
  
         -b, --batch
                Use "batch" mode.  In this mode, all input from the terminal is ignored.  Interrupt
                characters (such as ^C and ^\) still have an effect.  This is the default on a dumb
                terminal, or when the output is not a terminal.
  
         -c, --show-command
                Show the command name for each process. Default is to show the full  command  line.
                This option is not supported on all platforms.
  
         -i, --interactive
                Use  "interactive"  mode.   In  this  mode,  any  input  is  immediately  read  for
                processing.  See the section on "Interactive Mode" for an explanation of which keys
                perform   what  functions.   After  the  command  is  processed,  the  screen  will
                immediately be updated, even if the command was not understood.  This mode  is  the
                default when standard output is an intelligent terminal.
  
         -n, --non-interactive
                Use "non-interactive" mode.  This is indentical to "batch" mode.
  
         -q, --quick-mode
                Renice  pg_top to -20 so that it will run faster.  This can be used when the system
                is being very sluggish to improve the possibility of discovering the problem.  This
                option can only be used by root.
  
         -r, --remote-mode
                Monitor a remote database where the database is on a system other than where pg_top
                is running from.  pg_top will monitor a remote database if it  has  the  pg_proctab
                extension installed.
  
         -u, --show-uid
                Do  not  take the time to map uid numbers to usernames.  Normally, pg_top will read
                as much of the file "/etc/passwd" as is necessary to map all the user id numbers it
                encounters  into  login  names.   This  option  disables  all  that, while possibly
                decreasing execution time.  The uid numbers are displayed instead of the names.
  
         -V, --version
                Write version number  information  to  stderr  then  exit  immediately.   No  other
                processing  takes  place  when  this  option  is  used.   To  see  current revision
                information while pg_top is running, use the help command "?".
  
         -s TIME, --set-delay=TIME
                TIME Set the delay between screen updates  to  TIME  seconds.   The  default  delay
                between updates is 5 seconds.
  
         -o FIELD, --order-field=FIELD
                Sort  the  process display area on the specified field.  The field name is the name
                of the column as seen in the output, but in lower case.  Likely values  are  "cpu",
                "size",  "res", and "time", but may vary on different operating systems.  Note that
                not all operating systems support this option.
  
         -x COUNT, --set-display=COUNT
                Show only count displays, then exit.  A display is considered to be one  update  of
                the  screen.  This option allows the user to select the number of displays he wants
                to see before pg_top automatically exits.   For  intelligent  terminals,  no  upper
                limit is set.  The default is 1 for dumb terminals.
  
         -z USERNAME, --show-username=USERNAME
                Show  only  those  processes owned by USERNAME.  This option currently only accepts
                usernames and will not understand uid numbers.
  
         -h HOST, --host=HOST
                Specifies the host name of the machine on which the server is running. If the value
                begins  with  a  slash, it is used as the directory for the Unix domain socket. The
                default is taken from the PGHOST environment variable, if set.
  
         -p PORT, --port=PORT
                Specifies the TCP port or local Unix domain socket  file  extension  on  which  the
                server  is  listening for connections. Defaults to the PGPORT environment variable,
                if set.
  
         -U USERNAME, --username=USERNAME
                User name to connect as.
  
         -W, --password
                Force pg_top to prompt for a password before connecting to a database.
  
         Both COUNT and NUMBER fields can be specified as  "infinite",  indicating  that  they  can
         stretch  as  far  as  possible.   This  is  accomplished by using any proper prefix of the
         keywords "infinity", "maximum", or  "all".   The  default  for  count  on  an  intelligent
         terminal is, in fact, infinity.
  
         The  environment  variable  PG_TOP  is  examined  for  options  before the command line is
         scanned.  This enables a user to set his or her own defaults.  The number of processes  to
         display can also be specified in the environment variable PG_TOP.  The options -C, -I, and
         -u are actually toggles.  A second specification of any of these options will  negate  the
         first.   Thus  a  user  who  has  the  environment variable PG_TOP set to "-I" may use the
         command "top -I" to see idle processes.
  
  #### INTERACTIVE MODE
         When pg_top is running in "interactive mode", it reads commands from the terminal and acts
         upon them accordingly.  In this mode, the terminal is put in "CBREAK", so that a character
         will be processed as soon as it is typed.  Almost always,  a  key  will  be  pressed  when
         pg_top  is  between displays; that is, while it is waiting for time seconds to elapse.  If
         this is the case,  the  command  will  be  processed  and  the  display  will  be  updated
         immediately thereafter (reflecting any changes that the command may have specified).  This
         happens even if the command was incorrect.  If a key is pressed while  pg_top  is  in  the
         middle  of  updating  the display, it will finish the update and then process the command.
         Some commands require additional information, and the user will be  prompted  accordingly.
         While typing this information in, the user's erase and kill keys (as set up by the command
         stty) are recognized, and a newline terminates the input.
  
         These commands are currently recognized (^L refers to control-L):
  
         ^L     Redraw the screen.
  
         A      Display the actual query plan  (EXPLAIN  ANALYZE)  of  the  currently  running  SQL
                statement by re-running the SQL statement (prompt for process id.)
  
         C      Toggle the use of color in the display.
  
         c      Toggle the display of the full command line.
  
         d      Change  the  number of displays to show (prompt for new number).  Remember that the
                next display counts as one, so typing d1 will make pg_top show  one  final  display
                and then immediately exit.
  
         h or ? Display  a  summary of the commands (help screen).  Version information is included
                in this display.
  
         E      Display re-determined execution plan (EXPLAIN) of the SQL statement  by  a  backend
                process (prompt for process id.)
  
         e      Display  a  list  of  system  errors  (if any) generated by the last kill or renice
                command.
  
         i      (or I) Toggle the display of idle processes.
  
         k      Send a signal ("kill" by default) to a list of processes.  This acts  similarly  to
                the command kill(1)).
  
         L      Display the currently held locks by a backend process (prompt for process id.)
  
         M      Order by memory utilization.
  
         N      Sort by process id.
  
         n or # Change the number of processes to display (prompt for new number).
  
         o      Change  the order in which the display is sorted.  This command is not available on
                all systems.  The sort key names when viewing processes vary fron system to  system
                but  usually  include:   "cpu",  "res",  "size", "time".  The default is cpu.  When
                viewing   user   table   statistics:   "seq_scan",   "seq_tup_read",    "idx_scan",
                "idx_tup_fetch",  "n_tup_ins",  "n_tup_upd", "n_tup_del".  The default is seq_scan.
                When viewing user index statistics:  "idx_scan",  "idx_tup_fetch",  "idx_tup_read".
                The default is idx_scan.
  
         P      Sort by processor utilization.
  
         Q      Display the currently running query of a backend process (prompt for process id.)
  
         q      Quit pg_top.
  
         R      Display user table statistics.
  
         r      Change  the  priority  (the "nice") of a list of processes.  This acts similarly to
                the command renice(8)).
  
         s      Change the number of seconds to delay between displays (prompt for new number).
  
         T      Order by time.
  
         t      Toggle between cumulative or differential statistics when  viewing  user  table  or
                user index statistics.
  
         u      Display  only processes owned by a specific username (prompt for username).  If the
                username specified is simply "+", then processes belonging to  all  users  will  be
                displayed.
  
         X      Display user index statistics.
  
  #### THE DISPLAY
         The  actual  display  varies depending on the specific variant of Unix that the machine is
         running.  This description may not exactly match what is seen by pg_top  running  on  this
         particular machine.  Differences are listed at the end of this manual entry.
  
         The  top  few lines of the display show general information about the state of the system,
         including the last process id assigned to a process (on  most  systems),  the  three  load
         averages,  the  current time, the number of existing processes, the number of processes in
         each state (sleeping, running, starting, zombies, and stopped), and a percentage  of  time
         spent  in  each  of the processor states (user, nice, system, and idle).  It also includes
         information about physical and virtual memory allocation.
  
         The remainder of the screen displays information about individual processes.  This display
         is  similar  in  spirit to ps(1) but it is not exactly the same.  The columns displayed by
         pg_top will differ slightly between operating systems.  Generally,  the  following  fields
         are displayed:
  
         PID    The process id.
  
         USERNAME
                Username  of  the  process's  owner  (if  -u  is  specified,  a  UID column will be
                substituted for USERNAME).
  
         PRI    Current priority of the process.
  
         NICE   Nice amount in the range -20 to 20, as established by the use of the command nice.
  
         SIZE   Total size of the process (text, data, and stack) given in kilobytes.
  
         RES    Resident memory: current amount of process memory that resides in physical  memory,
                given in kilobytes.
  
         STATE  Current state (typically one of "sleep", "run", "idl", "zomb", or "stop").
  
         TIME   Number of system and user cpu seconds that the process has used.
  
         CPU    Percentage of available cpu time used by this process.
  
         COMMAND
                Name of the command that the process is currently running.
  
</details>
