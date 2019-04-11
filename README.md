# Linux scripts for running SAS

This repository contains Korn shell scripts used to automate various [SAS](https://www.sas.com/) functions and to standardize log checks.

## "Driver" scripts

KSH scripts are useful for running several related SAS programs from start to finish.  This ensures that the set of programs are run in the proper sequence.

Access to the toolbox from any terminal is provided by editing the user's `.bashrc` file **once** by running the following command:

```sh
echo 'export PATH=$PATH:/sae/admin/toolbox' >> ~/.bashrc
```

### Example

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

The above example Korn Shell script runs the 4 SAS program in sequence that perform the [SAHIE](https://www.census.gov/programs-surveys/sahie.html) Single Year processing.

Some details to note:

- The first line `#!/bin/ksh` is required – it indicates that the file is a Korn Shell Script.
- Comments are created using the `#` symbol (except as seen on the first line).
- [`do_one_sas_batch`](do_one_sas_batch) is a custom written Korn Shell program that runs the program but then checks the `.log` for about 15 problematic phrases.
- [`check_all_logs`](check_all_logs) checks the log files one more time and saves the output for record keeping.
- A `.ksh` file must have 'eXecution' privileges to run.  See `chmod` command documentation.

## `sas_log_check` & `do_one_sas_batch` scripts

These Korn shell scripts can either be run from another `.ksh` script or run directly from the command line.  Its main advantage is that it automatically checks the `.log` file for known SAS problems each time.

## Checks

These scripts include what we felt is the best set of things to check. This script is called by various other "flavors" of SAS scripts so that only one version needs to be maintained. The  following are the items currently (July 2015) checked:

- SEVERE Checks: Must be corrected
- ERROR: an error, typically means the program did not complete
- warning: common sign of a critical problem
- uninit: For Uninitialized
- truncate: Especially when reading a `.txt` or CSV file (ASCII – Flat File)
- repeats of BY values: Merge probably giving incorrect results

### Highly problematic checks

Should be corrected, should be 'rare'.

- Invalid argument
- End of line
- does not exist
- Lost card
- Missing values
- Multiple lengths
- Not resolved
- Invalid numeric data: Custom flag that will show up in log checker, must be written into program

### Low items checked

- converted
- Character values
- No observations: This might be okay – dataset of bad records should empty.
- 0 observations: Same as No observations

"Does not exist" during a `proc append` example:

- NOTE: Appending `WORK.C_RECODE_POV_SE` to `WORK.C_APPEND_POV_SE`.
- NOTE: BASE data set does not exist. DATA file is being copied to BASE file.

## Scripts

See the script header for details.

- [`check_all_logs`](check_all_logs) - runs the log check for every SAS program in the current directory and saves output to a permanent file
- [`do_one_sas_batch`](do_one_sas_batch) - used to execute the SAS program in batch mode and then check the log for any problems, SAS command line options can be passed after the program name
- [`sas_log_check`](sas_log_check) - searches for various messages in the SAS log that indicate a problem
