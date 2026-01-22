# LS_software
Software collection by Leonie Olivia Scheidweiler.
This collection is used to analyze satellite measurements of greenhouse gas sources.
The software is located in submodules within this repository.
When you click on them, you may have to manually navigate to the main branch.

## Setup
This repository assumes that you are working on the bwForCluster Helix and have access to SDS@hd.
Set this up according to the instructions on the respective websites
- [bwForCluster Helix](https://wiki.bwhpc.de/e/Helix/Getting_Started)
- [SDS@hd](https://sds-hd.urz.uni-heidelberg.de/management/index.php)
- Further instructions are available in the internal [IUPEDIA](https://iupedia.iup.uni-heidelberg.de:49200/index.php/BwForCluster_HELIX)

I suggest creating a symlink to the SDS in your home directory on the Helix Cluster for easy access, in my case using
```
ln -s ~/sds /mnt/sds-hd/sd18b002/users/lscheidweiler
```

I suggest always using a terminal multiplexer like tmux or screen, which allows you to detach while the scripts are doing their thing (especially RemoTeC).

Clone this repository into `~/sds` using
```
cd ~/sds/
git clone --recurse-submodules git@github.com:ATMO-IUP-UHEI/LS_software.git
```

Create directories for the data
```
mkdir ~/sds/data
mkdir ~/sds/data/raw_downloads  # contains downloaded spectra and cross-sections in a form not ready for use
mkdir ~/sds/data/input  # contains auxiliary information in a format that can be read
mkdir ~/sds/data/scenarios  # contains the satellite scenes
```

I will refer to `~/sds/data/scenarios/` as `$SCENARIO_DIR`.
I will refer to `~/sds/LS_software/` as `$SOFTWARE_DIR`.
You may replace these with your own locations.

## Workflow for using RemoTeC
These steps have to be run once, assuming you do not change anything major.
- Build RemoTeC using the instructions in [RemoTeC](https://github.com/ATMO-IUP-UHEI/LS_RemoTeC/tree/main).
- Create reference solar spectrum using the instructions in [PreProc_SOLAR](https://github.com/ATMO-IUP-UHEI/LS_PreProc/tree/main/PreProc_SOLAR).
- Create cross-section databases using the instructions in [PreProc_XSDB](https://github.com/ATMO-IUP-UHEI/LS_PreProc/tree/main/PreProc_XSDB).
- Setup accounts and API tokens for ASTER, ERA5, and EGG4 according to the instructions in [PreProc_ATM](https://github.com/ATMO-IUP-UHEI/LS_PreProc/tree/main/PreProc_ATM). These data sources are currently hardcoded.
- Create `settings_RTC_retrieve.nml`. I will refer to this as `$RTC_SETTINGS`.

These steps have to be run for every scenario.
Running RemoTeC in parallel is automated using [RemoTeC_run](https://github.com/ATMO-IUP-UHEI/LS_RemoTeC_run/tree/main).
Assume we are analyzing a new scenario with name `$SCENARIO_NAME`.
- First, download satellite data according to the instructions in [PreProc_L1B](https://github.com/ATMO-IUP-UHEI/LS_PreProc/tree/main/PreProc_L1B) and place them, for example, in `~/sds/data/raw_downloads/instrument/filename.zip`. I will refer to these file(s) as `$RAW_L1B_INPUT`.
```
cd $SCENARIO_DIR/
$SOFTWARE_DIR/PreProc/create_rundir.sh $SCENARIO_NAME
cd $SCENARIO_NAME/
cp $RAW_L1B_INPUT tmp/spectra/
sbatch --wait $SOFTWARE_DIR/PreProc/PreProc_L1B/slurm_job.sh enmap  # check the path in here, replace your instrument name
python3 $SOFTWARE_DIR/PreProc/PreProc_ATM/download_scripts/download_aster.py
python3 $SOFTWARE_DIR/PreProc/PreProc_ATM/download_scripts/download_era5.py
python3 $SOFTWARE_DIR/PreProc/PreProc_ATM/download_scripts/download_egg4.py
sbatch --wait $SOFTWARE_DIR/PreProc/PreProc_ATM/slurm_job.sh  # check the path in here
python3 $SOFTWARE_DIR/PreProc/create_lst.py
cp $RTC_SETTINGS INI/
python3 $SOFTWARE_DIR/RemoTeC_run/retrieve.py  # check settings.ini in the same folder.
```
You may now detach from the tmux session and check the progress using `squeue`.

## Workflow for using Matched Filter
These steps have to be run once, assuming you do not change anything major.
- Generate unit absorption spectra according to the instructions in [Matched_Filter](https://github.com/ATMO-IUP-UHEI/LS_matched_filter/tree/main).

These steps have to be run for every scenario.
- This workflow assumes that the RemoTeC folder structure is created and the L1B preprocessor has been executed.
```
cd $SCENARIO_DIR/$SCENARIO_NAME/
python3 $SOFTWARE_DIR/matched_filter/run/matched_filter.py
```
