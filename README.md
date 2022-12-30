# APTAAnomaly-bin

This repository contains releases for the APTAAnomaly binaries. Both the dashboard and the preprocessing binaries are here.

# APTAAnomaly

APTAAnomaly can be used to preprocess log files (currently only evtx files are supported, more to come) and apply our algorithms to them to calculate novelty scores. These scores intend to show how novel observed behavior is. A high score means that the according behavior has not been seen before often.

### Usage

```
./aptaanomaly --help
usage: aptaanomaly [-h] --input INPUT [--output OUTPUT] [--output-type {dashboard,velociraptor}] [--flexfringe FLEXFRINGE]
                   [--sortscores] [--logfile LOGFILE] [--skip-existing] [--cpuload {0.0 - 1.0}] [-v]

options:
  -h, --help            show this help message and exit
  --input INPUT         input evtx file path
  --output OUTPUT       output file path
  --output-type {dashboard,velociraptor}
                        output format type, defaults to "dashboard"
  --flexfringe FLEXFRINGE
                        Path to flexfringe executable
  --sortscores          Sort the output files by the score column
  --logfile LOGFILE     File to write logs to
  --skip-existing       Skip existing files instead of overwriting them
  --cpuload {0.0 - 1.0}
                        fraction of cores to use
  -v                    Enable verbose logging
```

### Notes
On windows, it is not necessary to specify an input path if you intend to use the evtx files from the machine you are running aptaanomaly on, as `--input` defaults to `C:\windows\system32\winevt\Logs`

It is also not necessary anymore to specify a path to the flexfringe binary anymore, as it is bundled with the executables now. The `--flexfringe` option is still there in case you need to override it.

It is now also autodetected whether a dataset contains a single host, or multiple hosts. If `--input` points to a directory that directly contains evtx files, it is interpreted as a single host dataset and the hostname will be set to the hostname of the machine that is running aptaanomaly. If `--input` points to a directory with subdirectories containing evtx files, it is interpreted as a multihost dataset and the names of the subdirectories will be used as hostnames.

To clarify, a multi host dataset looks like this:
```
multi-host
├── host-1
│   ├── 1.evtx
│   ├── 2.evtx
│   └── 3.evtx
└── host-2
    ├── 1.evtx
    ├── 2.evtx
    └── 3.evtx
```

And a single host like this:
```
single-host
├── 1.evtx
├── 2.evtx
└── 3.evtx
```
### Examples

To collect and process the evtx files belonging to the current machine (windows only), for use with the dashboard later on:
```
.\aptaanomaly-windows.exe --output C:\path\to\output\dir
```

To process previously collected evtx files:
```
.\aptaanomaly-windows.exe --input C:\path\to\evtx\files --output C:\path\to\output\dir
```

# Dashboard

The dashboard is used to view the calculated novelty scores, per host, per log file, in a convenient way. It also allows importing findings from other tools (currently supported: chainsaw) and display them in the timelines.

### Usage

```
./dashboard --help
usage: dashboard [-h] [--preprocessed [PREPROCESSED ...]] [--logpath [LOGPATH ...]] [--noveltypath [NOVELTYPATH ...]] [--chainsaw [CHAINSAW ...]] [--evtx [EVTX ...]]
                 [--ff FF] [--db DB] [--builddb] [--experimental]

options:
  -h, --help            show this help message and exit
  --preprocessed [PREPROCESSED ...]
                        Path to preprocessed input folder with novelty and logs
  --logpath [LOGPATH ...]
                        Path to the log entries to show in table (legacy), use --preprocessed
  --noveltypath [NOVELTYPATH ...]
                        Path to the novelty scores for each log entry (legacy), use --preprocessed
  --chainsaw [CHAINSAW ...]
                        Chainsaw json input file, to add annotations
  --evtx [EVTX ...]     Input folder containing evtx files
  --ff FF               Path to flexfringe executable
  --db DB               Database connection string
  --builddb             If set, (re)build database
  --experimental        enable experimental (potentially unfinished) features
  ```


To get started, you need to build a database with your input files. 

### Examples

To start the dashboard with previously preprocessed (with aptaanomaly) input files:
```
./dashboard --preprocessed /path/to/preprocessed/input --builddb
```

Or directly from evtx files:
```
./dashboard --evtx /path/to/evtx/files --builddb
```

If you want to include chainsaw findings as annotations, first run chainsaw on your evtx files and tell it to give you json output:

```
./chainsaw hunt /path/to/evtx -s sigma --mapping mappings/sigma-event-logs-all.yml --json --output /path/to/chainsaw.json

```

Then:
```
./dashboard --evtx /path/to/evtx/files --chainsaw /path/to/chainsaw.json --builddb
```
This would also work fine with `--preprocessed`, as long as chainsaw is ran on the same evtx files as aptaanomaly, since it needs to match the chainsaw findings to the corresponding evtx log lines.