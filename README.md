# Ceph RGW Gap List Tool

This project provides systemd unit and timer files to automate the execution of a tool that lists gaps in Ceph RGW (RADOS Gateway) buckets. The tool helps identify inconsistencies or missing objects in RGW buckets.

The `rgw-gap-list.py` tool is designed to:
- Identify gaps in Ceph RGW bucket data
- Perform consistency checks across buckets
- Generate reports on bucket scrub metadata
- Support multi-run/multi-host environments for large-scale deployments

This tool is particularly useful for maintaining data integrity in Ceph RGW systems by detecting missing or inconsistent objects across buckets.

## Systemd Unit Files

### Service File: `rgw-gap-list@.service`

The service file defines how the gap list tool should be executed.


### Timer File: `rgw-gap-list@.timer`

The timer file defines when the service should run.
The default schedule is set to run on the 1st and 15th of each month at 04:00:00.

## Usage

### Enabling and Starting the Timer

After copying all files into place, you can enable and start the timer as follows:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now rgw-gap-list@1.timer
```

**NOTE:** The enable line can be ran multiple times, incrementing the `1` value to set up multiple processes to run on a single host.

### Checking Timer Status

```bash
sudo systemctl status rgw-gap-list@1.timer
```

### Viewing Logs

```bash
sudo journalctl -u rgw-gap-list@1.service
```

## Configuration

The tool uses a configuration file at `/etc/default/rgw-gap-list` which contains command-line options for the Python script.

Example configuration:
```bash
# Basic Ceph configuration
OPTS="-c /etc/ceph/ceph.conf"

# Example with specific buckets and verbosity
OPTS="-b 'bucket1 bucket2' -vv"

# Example with custom pools and namespace
OPTS="-p 'my_pool1 my_pool2' --namespace my-namespace"
```

The available options include:
- `-c CONF`: Ceph conf file to use, default '/etc/ceph/ceph.conf'
- `-b BUCKETLIST`: Bucket(s) to operate on (space-separated list)
- `-e EXCLUDELIST`: Bucket(s) to skip (space-separated list)
- `-f EXCLUDEFILE`: File with list of buckets to skip
- `-p POOL`: Bucket Data Pool(s) to use
- `-s SYNCPOOL`: Synchronization pool to use
- `-v`: Verbosity level (multiple -v's for higher verbosity - up to 3 maximum)
- `-r`: Generate bucket scrub metadata report
- `-g`: Dump gap results from RADOS object contents
- `-x`: Dump verified results from RADOS object contents
- `--namespace NAMESPACE`: Namespace to use for sync/results objects

The default configuration sets `OPTS=""` which means all defaults are used.

## Customizing the Schedule

The timer can be customized by modifying the `OnCalendar` directive in the `.timer` file. Several examples are included:
- Monthly: Run on the 1st of each month
- Every two weeks: Run an hour after boot and then every two weeks
- Weekly: Run every Monday at midnight

## Requirements

- Ceph RGW environment with proper access credentials
- Python script (`rgw-gap-list.py`) installed in `/usr/local/bin/`
- Systemd service manager

## Notes

This tool is designed to run as a scheduled task to periodically check for gaps in Ceph RGW buckets, helping confirm data integrity and consistency across the system.