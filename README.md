# Synobackup Docker Interrupt
A Python script that stops and starts the Docker containers while Synology backups are running.

**Rationale**: Synology supplies a tool called *Hyper Backup*, which unfortunately does not integrate well with containers.
Indeed, using *Hyper Backup* to back up running containers may lead to saving locked databases or in-use files.
Unfortunately, *Hyper Backup* does not provide any triggers to execute tasks before starting and after ending, where containers may typically be stopped and started.

This script fills the gap when executed before the backup operation using a scheduled task by performing the following actions:

1. Stop the running containers,
2. Wait an *initial amount of seconds*, giving the time for *Hyper Backup* to kick in,
3. Check periodically whether the backup operation is still occurring,
4. After the backup operation, restart the previously running containers.

## Getting the script
The script comes in two flavors for simplicity.
Head over to the [releases page](https://github.com/JamesMenetrey/synobackup-docker-interrupt/releases) for downloading.

- Packaged as a single folder, which includes all the dependencies and the Python interpreter,
- Packaged as a single file, which includes all the dependencies and the Python interpreter.

### Single-folder option
If you don't mind having a folder with many scripts and dependencies, prefer this option.
Extract the content of the *synobackup_docker_interrupt_singlefolder* archive on your Synology NAS.
The execution of the program is as follows:

```bash
./synobackup_docker_interrupt
```

### Single-file option
If you would like a clean single-file program, prefer this option, but mind the drawbacks of the [single-file approach of Pyinstaller](https://pyinstaller.org/en/stable/operating-mode.html#how-the-one-file-program-works) (the tool used to package the script).
The script must specify a temp folder to deflate the embedded dependencies, which is not `/tmp` because Synology OS does not mount that path with the executable bit.
The execution of the program is as follows:

```bash
./synobackup_docker_interrupt --runtime-tmpdir <temp_path>
```

## Usage
```
synobackup_docker_interrupt, ver.1.2.0. Written by Jämes Ménétrey.
Usage: singlefolder/synobackup_docker_interrupt/synobackup_docker_interrupt <initial_backup_check> <interval_backup_check>
        - initial_backup_check: time [s] to check whether a backup is occurring for the first time.
        - interval_backup_check: time interval [s] to check whether a backup occurs after the first time.
The exit code is zero when the script completes successfully, otherwise non-zero.

Note: initial_backup_check is used after the containers are stopped, while interval_backup_check is
      used on a regular basis once the first check has occurred. This allows for planning a shutdown of the
      containers with a more significant interval before the backup operation occurs.
```

## Packaging from sources
Docker is used to create both single-folder and single-file packages.
This is achieved by executing the helper script `package.sh`.

## Changelog

- v1.2.0: Use the Synology OS API to start and stop the containers, preventing from receiving a notification for each stopped container.
- v1.1.0: Returns a non-zero exit code in case a Docker container cannot be stopped or restarted.
- v1.0.0: Initial release.