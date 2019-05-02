# Linux scripts for running SAS

This repository contains Korn shell scripts used to automate various [SAS](https://www.sas.com/) functions and to standardize log checks.

## Setup

The shell scripts need the extension `.ksh` and should be stored in a central location accessible by all users. By doing so, only one copy of each script needs to be maintained. Then, access to the scripts from any terminal is provided by editing the user's `.bashrc` file **once** by running the following command:

```sh
echo 'export PATH=$PATH:/your_path' >> ~/.bashrc
```

Users may need to log out and back in to the server for this change to take effect. Once ready, a SAS program can be executed in batch mode:

```batch
do_one_sas_batch.ksh my_program.sas
```

## "Driver" scripts

Shell scripts are useful for running several related SAS programs from start to finish.  This ensures that the set of programs are run in the proper sequence.

### Example

Unix/Linux has several scripting languages available to bundle several commands into a single command (Windows only has 1:  DOS).  For no particular reason, we decided to use Korn Shell (.ksh) scripts.

```ksh
#!/bin/ksh
# 00_sahie_dissemination_singleyear_driver.ksh
#
#  to run type ./{progname}.ksh (after making priv executable)

# set -e halts execution if an error is encountered
set -e

do_one_sas_batch.ksh 01_making_sahie_singleyear.sas
do_one_sas_batch.ksh 02_qc_sahie_singleyear.sas
do_one_sas_batch.ksh 03_write_sahie_singleyear_txt.sas
do_one_sas_batch.ksh 04_write_sahie_singleyear_csv.sas

check_all_logs.ksh
```

The above example Korn Shell script runs the 4 SAS programs in sequence that perform the [SAHIE](https://www.census.gov/programs-surveys/sahie.html) single year dissemination processing.

Some details to note:

- The first line `#!/bin/ksh` is required – it indicates that the file is a Korn Shell Script.
- Comments are created using the `#` symbol (except as seen on the first line).
- [`do_one_sas_batch`](do_one_sas_batch) is a custom written Korn Shell program that runs the program but then checks the `.log` for about 15 problematic phrases.
- [`check_all_logs`](check_all_logs) checks the log files one more time and saves the output in one file for record keeping.
- A `.ksh` file must have 'eXecution' privileges to run.  See `chmod` command documentation.

## Log Checks

The `sas_log_check` script includes what we felt is the best set of things to check. This script is called by various other "flavors" of SAS scripts so that only one version needs to be maintained. The following are the items currently (Sept 2018) checked:

### Severe checks

Must be corrected

- ERROR: a SAS error, typically means the program did not complete (case sensitive)
- Warning: common sign of a critical problem, common exceptions are ignored
- Uninit: For Uninitialized
- Truncate: Especially when reading a `.txt` or CSV file (ASCII – Flat File)
- Repeats of BY values: Merge probably giving incorrect results
- Fatal: rare SAS execution error

### High priority checks

Should be corrected, should be 'rare'

- Invalid argument
- End of line
- Does not exist (ignores proc append note)
- Lost card
- Missing values
- Multiple lengths
- Not resolved
- Invalid numeric data
- Invalid data
- Custom note: Custom flag that will show up in log checker, must be written into program

### Low priority checks

- Converted
- Character values
- No observations: This might be okay – dataset of bad records should empty.
- 0 observations: Same as No observations
- _Error_
- Division by zero
- A missing value
- Not permitted

## Scripts

These Korn shell scripts can either be run from another `.ksh` script or run directly from the command line.  Their main advantage is that it automatically checks the `.log` file for known SAS problems each time.

See the script header for details.

- [`sas_log_check`](sas_log_check) - searches for various messages in the SAS log that indicate a problem
- [`do_one_sas_batch`](do_one_sas_batch) - used to execute the SAS program in batch mode and then check the log (by executing `sas_log_check`) for any problems, SAS command line options can be passed after the program name
- [`check_all_logs`](check_all_logs) - runs the `sas_log_check` for every SAS program in the current directory and saves output to a permanent file

## `SAS_Checker_list.csv`

This is machine readable CSV created to assist alternative log checking methods. It can be used as the data in a data driven log checking program.
