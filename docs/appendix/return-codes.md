# Return Codes

Return codes, sometimes called exit codes, are numbers returned from the `rig` executable 
to indicate the status of the requested command at exit. These codes can be interpreted by 
scripts calling `rig` to react to success or failure states. Generally a return code 
of `0` is used to indicate success. Non-zero return codes communicate that an error has 
occurred and give an idea of the error type through the value returned.

`rig` exit code values are:

* `1` - Default
  * Returned when an uncaught error is raised and the program exited unexpectedly. This is
  used by the underlying cli framework as well. Project scripts may also return this code.
* `12` - Environmental 
  * Returned when there is a problem with the execution environment and 
  the current command cannot continue. This is usually the result of the
  Docker Machine not running or when Docker is not configured correctly.
  Backup and restore errors generally fall in this category too.
* `13` - External Command / Upstream 
  * Returned when external scripts or setup commands have reported failure.
