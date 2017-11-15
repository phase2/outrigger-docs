# Return Codes

Return codes, sometimes called exit codes, are numbers returned from the `rig` executable 
to indicate the status of the command when on exit.  Exit codes can be interpreted by 
machine scripts to adapt in the event of successes of failures. Generally a return code 
of `0` is used to indicate success.  Non-zero return codes are used to communicate that
and error has occurred and can give an idea of what type of error was raised.

`rig` exit codes used to indiciate certain conditions and are as follows:

* `1` - Unexpected
  * Returned when an uncaught error is raised and the program exited unexpectedly
* `12` - Environmental 
  * Returned when there is a problem with the execution environment and 
  the current command cannot continue. This is usually the result of the
  Docker Machine not running or Docker is not configured correctly. Backup
  and restore errors generally fall in this category too.
* `13` - External Command / Upstream 
  * Returned when external scripts or setup commands have reported failure.
