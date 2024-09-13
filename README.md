# htcondor-ide-tools
Tools for using HTCondor with IDEs.

These tools strive to integrate IDEs with [HTCondor](https://htcondor.org/), for example by allowing remote editing via jobs running on an HTC cluster directly from the IDE of your choice.

DISCLAIMER: This project is fully open source, and not affiliated with one of the listed IDEs or the HTCondor project itself.

## Visual Studio Code

The `vscode/vscode_condor_ssh` script can be used to edit your code inside HTCondor cluster jobs, to inspect results, run Jupyter Notebooks interactively and more.

### Limitation and requirements
It requires VS Code to be executed from an Access Point, and the HTCondor Python API to be installed.

The usage instructions assume that the mentioned script is placed in the path.

### Usage (admin point of view)
From the admin perspective, it might be of interest to apply special policies to IDE jobs, i.e. route them to dedicated / reserved resources, apply runtime limits etc. To ease this, the provided script sets a ClassAd attribute `EditorJob` for all submitted jobs (can be changed at the top of the script).

You may for example apply a policy such as:
```
RemoveEditorJobWallTime = ( (EditorJob =?= True) && ( RemoteWallClockTime > 1 * 24 * 60 * 60 ) )
SYSTEM_PERIODIC_REMOVE = $(SYSTEM_PERIODIC_REMOVE) || $(RemoveEditorJobWallTime)
```
on the access points to limit the maximum wall clock time used by the job to 24 hours, or have slots dedicated to such jobs on your execute points. You may also want to set an appropriate `SUBMIT_REQUIREMENT`.

### Usage (user point of view)
1. Make sure you have the `Remote SSH`-Plugin installed (full name: `ms-vscode-remote.remote-ssh`).
2. You need to adapt a few configurations. Click on the "gear" button in the bottom left corner to edit your settings. Then search for `ssh` and adapt the following options:
   ```
   Remote.SSH: Path => set the value:
       vscode_condor_ssh
   ```
   If you encounter issues at some point, you may also want to set the following, which will show more information during the remote connection:
   ```
   Remote.SSH: Show Login Terminal→ Enable this
   ```
   You might also want to set and extend:
   ```
   Remote.SSH: Default Extensions
   ```
   at some point to contain the extensions which should be installed automatically in any remote session (such as `ms-toolsai.jupyter` for Jupyter Notebook support).
   
   In case you prefer to set these settings via JSON, i.e. "Preferences: Open User Settings (JSON)", you can use these lines (they include the optional settings from above):
   ```
   {
       "remote.SSH.path": "vscode_condor_ssh",
       "remote.SSH.defaultExtensions": [
           "ms-toolsai.jupyter"
       ],
       "remote.SSH.showLoginTerminal": true
   }
   ```
3. Finally, you'll need a JDL file with the requirements for your job. This could for example look as follows:
   ```
   ~/JDLs/IDE_vscode.jdl
   Transfer_input_files    = .ssh
    
   Request_cpus = 1
   Request_memory = 1000 MB
   Request_disk = 1000 MB
    
   Queue
   ```
   An executable or arguments are not required, but depending on your HTCondor configuration, you may of course have to specify some additional classad attributes.

   In case you use relative paths inside the JDL file, such as the `.ssh` directory mentioned above, please note that they are resolved from the working directory of VS Code! This means that in case you start VS Code in your home directory, the files are expected to be placed relative to your home directory.

With this, the initial setup is complete.

From now on, inside VS Code, you can open a remote connection (most likely using `Ctrl+Shift+P` => `Remote SSH: Connect to Host`) and then enter the following "host name":
```
VS Code - Remote Host
JDL:~/JDLs/IDE_vscode.jdl
```
During your first time doing this, you will be asked what kind of operating system the remote host is running, which is `Linux`.

### Potential issues
In some cases, we observed error messages  in the startup logs of a new connection related to wget failing due to TLS errors, or later on VS Code getting stuck with `Downloading VS Code server`. These are sadly issues related to the Microsoft servers distributing these components when they are overloaded.

It might help to re-try in this case (just reload the VS Code windows and / or VS Code, no need to cancel the job) as timeouts are quite large. Once VS Code and the job are up and running, you are not affected by such issues anymore.

In case of problems, please run the command (via `Ctrl+Shift+P`):
```
Remote-SSH: Show Log
```
This should hopefully contain an error message.

### Important notes
- Exiting VS Code will not terminate the running job.
- This is quite helpful during daily usage, since VS Code may want to reconnect e.g. when being restarted, after some plugin installations or when opening different directories on the remote end. Reconnection works fine as long as the job is still running and you always start VS Code on the same machine.  
Please note that this also means you have to explicitly terminate the job (via `condor_rm <jobID>`) to free the allocated resources, otherwise they will be blocked for other users until the maximum runtime is exceeded!
- As these kinds of jobs are meant for interactive usage and usually terminate in "unhealthy" ways (e.g. runtime limit exceeded), there is no automatic copying back via Transfer_Output_Files .
- Please remember that in most clusters, the job working directory is scratch space, so usage of `Git` or other tooling is recommended.
- In some cases, you may observe a strange error like `Cannot read propoerties of undefined (reading 'after')` when trying to connect. This is usually a sign that VS Code does not understand your `.ssh/config`, please review the contents (and maybe first try without an SSH configuration to confirm this is really the cause). See also [this upstream issue](https://github.com/microsoft/vscode/issues/212685).

