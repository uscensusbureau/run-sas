# Linux scripts for running SAS

## Purpose

This documentation provides an overview of the usage of Korn shell scripts to automate various SAS functions and to standardize log checks.

## When

KSH scripts are useful for running several related SAS programs from start to finish.  This ensures that the set of programs are run in the proper sequence.

## What (Procedure)
The usage of a driver .ksh files **is suggested** but not mandatory. The usage of the [`do_one_sas_batch`](do_one_sas_batch) script for SAS programs **is required**.

Access to the toolbox from any terminal is provided by editing the user's `.bashrc` file **once** by running the following command:

```sh
echo 'export PATH=$PATH:/sae/admin/toolbox' >> ~/.bashrc
```

## An Example Korn Shell (.ksh) Script

Unix/Linux had several scripting languages available to bundle several commands into a single command (Windows only has 1:  DOS).  For no particular reason, we decided to use Korn Shell (.ksh) scripts.

```ksh
#!/bin/ksh
# 00_sahie_dissemination_singleyear_driver.ksh
#
#  to run type ./{progname}.ksh (after making priv executable)

do_one_sas_batch.ksh 01_making_sahie_singleyear
do_one_sas_batch.ksh 02_qc_sahie_singleyear
do_one_sas_batch.ksh 03_write_sahie_singleyear_txt
do_one_sas_batch.ksh 04_write_sahie_singleyear_csv

check_all_logs.ksh
```

The above example Korn Shell script runs the 4 SAS program in sequence that perform the SAHIE Single Year processing.

Some details to note:

- The first line `#!/bin/ksh` is required – it indicates that the file is a Korn Shell Script.
- Comments are created using the `#` symbol (except as seen on the first line).
- [`do_one_sas_batch`](do_one_sas_batch) is a custom written Korn Shell program that runs the program but then checks the `.log` for about 15 problematic phrases.
- [`check_all_logs`](check_all_logs) checks the log files one more time and saves the output for record keeping.
- A `.ksh` file must have 'eXecution' privileges to run.  See `chmod` command documentation.

## `sas_log_check` & `do_one_sas_batch` Korn Shell (.ksh) Script

This Korn shell script can either be run from another .ksh script or run directly from the command line.  Its main advantage is that it automatically checks the `.log` file for known SAS problems – EACH TIME.

The SAEB team worked with other branches to implement what we felt is the best set of things to check. This script is called by various other "flavors" of SAS scripts so that only one version needs to be maintained. The internal workings of the script are out of scope for this document but the following are the items currently (July 2015) checked:

- **SEVERE Checks - Must be corrected** – if not possible **Branch Chief Approval required**.
- ERROR: an error, typically means the program did not complete
- warning                common sign of a critical problem
- uninit                        For Uninitialized
- truncate                        Especially when reading a `.txt` or CSV file (ASCII – Flat File)
- repeats of BY values        Merge probably giving incorrect results

### **HIGHLY Problematic Checks** – Should be corrected  - Should be 'rare'.

if not possible note and get peer reviewer approval.

- Invalid argument
- End of line
- does not exist
- Lost card
- Missing values
- Multiple lengths
- Not resolved
- Invalid numeric data - Custom flag that will show up in log checker, must be written into program

### **LOW Items Checked**

**Allowable but should be researched as to the cause and validated by peer reviewer.**

- converted
- Character values
- No observations:                This might be okay – dataset of bad records should empty.
- 0 observations:                Same as No observations

"Does not exist" during a `proc append` Example:

- NOTE: Appending `WORK.C_RECODE_POV_SE` to `WORK.C_APPEND_POV_SE`.
- NOTE: BASE data set does not exist. DATA file is being copied to BASE file.

## Other toolbox scripts

Available scripts and a short description of each, see the script header for details:

- [`check_all_logs`](check_all_logs) - runs the log check for every SAS program in the current directory and saves output to a permanent file
- [`do_one_sas_batch`](do_one_sas_batch) - used to execute the SAS program in batch mode and then check the log for any problems, SAS command line options can be passed after the program name
- [`sas_log_check`](sas_log_check) - searches for various messages in the SAS log that indicate a problem
