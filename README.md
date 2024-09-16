# htcondor-ide-tools
Tools for using HTCondor with IDEs.

These tools strive to integrate IDEs with [HTCondor](https://htcondor.org/), for example by allowing remote editing via jobs running on an HTC cluster directly from the IDE of your choice.

DISCLAIMER: This project is fully open source, and not affiliated with one of the listed IDEs or the HTCondor project itself.

### Usage (admin point of view)
From the admin perspective, it might be of interest to apply special policies to IDE jobs, i.e. route them to dedicated / reserved resources, apply runtime limits etc. To ease this, the provided script sets a ClassAd attribute `EditorJob` for all submitted jobs (can be changed at the top of the script).

You may for example apply a policy such as:
```
RemoveEditorJobWallTime = ( (EditorJob =?= True) && ( RemoteWallClockTime > 1 * 24 * 60 * 60 ) )
SYSTEM_PERIODIC_REMOVE = $(SYSTEM_PERIODIC_REMOVE) || $(RemoveEditorJobWallTime)
```
on the access points to limit the maximum wall clock time used by the job to 24 hours, or have slots dedicated to such jobs on your execute points. You may also want to set an appropriate `SUBMIT_REQUIREMENT`.

## Visual Studio Code

The `vscode/vscode_condor_ssh` script can be used to edit your code inside HTCondor cluster jobs, to inspect results, run Jupyter Notebooks interactively and more.

More information can be found in [the dedicated README](vscode/README.md).
