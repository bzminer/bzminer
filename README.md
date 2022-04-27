## Why use BzMiner (v9.0.04)?
- Supported Algos:
    - Ethash (AMD, Nvidia)
    - Etchash (AMD, Nvidia) 
    - Kawpow (AMD, Nvidia)
    - Olhash (AMD, Nvidia, 1% dev fee)
    - Alephium (AMD, Nvidia)
    - Kaspa (AMD, Nvidia, 1% dev fee + 2% Kaspa dev team fee)
    - Ixian (AMD, Nvidia, 1% dev fee)
- Low dev fee of 0.5%
- Nvidia GPU memory temperature monitoring on Linux and Windows
- Improved LHR Strategy dual/multi coin mining (ethash+kawpow, ethash+ol, ethash+alph)
- Multi coin mining supports three different strategies; Parallel mining, Alternate mining, DAG gen only mining
- Can overclock per algorithm when alternating multi coin mining
- Set Overclocks PER multi mine algo!
- Control algorithm/pool/wallet mining PER device, as well as Multi-coin mining on a single GPU
- Awesome Easy to use Linux and Windows miner (GUI available through browser)
- Hung GPU detection
- Watchdog service which automatically restarts BzMiner if anything goes wrong (such as a hung gpu)
- Customizable display
- Memory junction temperature monitoring (windows only)
- HTTP API!
- Remote management of all mining rigs on your network
- Fast Cuda v11 and OpenCL miner
- Constantly being improved based on feedback from you!
- Ability to OC in miner (OC's are removed during DAG generation to prevent invalid DAG generation)
- Advanced mining features, including cooldown, ramp up, dual/multi coin mining
- DAG Validation for very high OC cards
- Auto intensity, dyanamically adjusts gpu workloads, reducing stales while keeping hashrate high
- Ubuntu 16.04 support

## Prerequisites
- Windows:
  - Requires Nvidia drivers supporting Cuda 11.2 or greater (Nvidia driver >= 460.27.03)
- Linux:
  - Nvidia driver >= 460.27.03
- Linux (Ubuntu 16.04):
  - Ubuntu 16.04 or higher
  - Cuda 11.2
  - Nvidia driver >= 495.29.05

## Mining OS's that include BzMiner
- MMPOS - https://app.mmpos.eu/
- HiveOS - https://hiveos.farm/


## Discord Server
https://discord.gg/NRty3PCVdB

## Official Website
http://www.bzminer.com


## Requirements
- At least one Nvidia GPU and Nvidia drivers
- Or at least one AMD GPU and AMD drivers

## Quick Start
Launch `bzminer` with the wallet and pool address as parameters:

```
bzminer -w 0x0000000000000000000000000000000000000000 -p ethstratum+tcp://eth.geodns.flexpool.io:4444 -r worker_name
```

You may provide more than one pool address to -p for fallback pools on network errors:
```
bzminer -w 0x0000000000000000000000000000000000000000 -p ethstratum+tcp://eth.geodns.flexpool.io:4444 stratum+tcp://usw-eth.hiveon.net:4444 -r worker_name
```

Optionally you can edit `config.txt` and launch `bzminer`. You can specify the config file with the `-c` argument:

```
bzminer -c eth-config.txt
```

### Ethereum

```
bzminer -a ethash -w 0x0000000000000000000000000000000000000000 -p stratum+tcp://usw-eth.hiveon.net:4444 stratum+tcp://us1.ethermine.org:4444 -r worker_name
```

### Ethereum Classic

```
bzminer -a etchash -w 0x0000000000000000000000000000000000000000 -p stratum+tcp://us1-etc.ethermine.org:4444 stratum+tcp://eu1-etc.ethermine.org:4444 -r worker_name
```

### Ravencoin

```
bzminer -a kawpow -w 0x0000000000000000000000000000000000000000 -p stratum+ssl://stratum-ravencoin.flypool.org:3443 -r worker_name
```

### Overline

```
bzminer -a olhash -w 0x0000000000000000000000000000000000000000 -p ethproxy+ssl://us.extremehash.net:3443 ethproxy+ssl://eu-ol.extremehash.net:3443 -r worker_name
```

### Alephium

Alephium mining may require 4 wallet addresses (solo mining)

For solo mining change "stratum" to "alphstratum"

```
bzminer -a alph -w 000000 -p stratum+tcp://eu.metapool.tech:20032 alphstratum+tcp://185.71.66.100:10159
```

### Kaspa

Kaspa mining currently requires a node running (experimental pool implementation provided in BzMiner)

For solo mining, use node+tcp

```
bzminer -a kaspa -w 000000 -p node+tcp://127.0.0.1:16110
```

For pool mining change "node" to "stratum"

```
bzminer -a kaspa -w 000000 -p stratum+tcp://acc-pool.pw:16061
```

### Ixian

Solo and pool both use the GetWork http/https protocol

```
bzminer -a ixi -w 000000 -p http://ixian.changeling.biz:8081
```

### Dual Mining (eth + alph)

```
bzminer -a ethash -w 000000 -p stratum+tcp://us1.ethermine.org:4444 --a2 alph --w2 000000 --p2 stratum+tcp://eu.metapool.tech:20032
```

You can change the multi mining type using the `multi_mine_type` option in the command line. By default its set to 0 for parallel mining, but can be set to 1 for alternate mining or 2 for mine only during DAG generation


## GUI
The GUI desktop app has been discontinued as of v4.3 in favor of the browser gui, which can be accessed by opening `index.html`, or by navigating to `http://your-rigs-ip:port/` in your favorite browser.

The GUI will display (and allow you to navigate to) other rigs running BzMiner. This is done through a UDP broadcast message which can be disabled on instances of BzMiner by setting "disable_udp" to true in the config file. 

![image](https://user-images.githubusercontent.com/83083846/147267304-8357dd99-1638-4d83-a41a-0ebabd01cd4b.png)


## CLI
BzMiner is a command line interface. Simply update `config.txt` and launch `bzminer`.

`bzminer` has an optional parameter, `-c`, which can be called to load a different configuration file. eg. `bzminer -c custom_config.txt`.

`bzminer` also has optional parameters for setting the pool url, wallet address, algorithm and rig/worker name for all devices

```
>bzminer --help
BzMiner - Advanced Crypto Miner
Usage: bzminer [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  --version                   Get the version of this binary
  -a TEXT                     Default Mining algorithm. eg. 'ethash'
  --a2 TEXT                   Default second Mining algorithm (dual mine). eg. 'ethash'
  -r TEXT                     Default Rig (worker/username) name. eg. 'Rig'
  --r2 TEXT                   Default Pool username for second algo (dual mine)
  --pool_password TEXT        Default Pool password
  --pool_password2 TEXT       Default Pool password for second algo (dual mine)
  --nvidia INT                Only mine with Nvidia devices (0 = false, 1 = true)
  --amd INT                   Only mine with AMD devices (0 = false, 1 = true)
  -o TEXT                     If provided, output will be logged to this file
  -w TEXT ...                 Wallet Address. If algorithm requires more than one address, list them same as -p
  --w2 TEXT ...               Wallet Address for second algo (dual mine). If algorithm requires more than one address, list them same as -p
  -p TEXT ...                 Array of Pool Addresses. eg. stratum+tcp://us1.ethermine.org:4444  stratum+tcp://us2.ethermine.org:4444
  --p2 TEXT ...               Array of Pool Addresses for second algo (dual mine). eg. stratum+tcp://us1.ethermine.org:4444  stratum+tcp://us2.ethermine.org:4444
  -v INT                      Set log verbosity. 0 = Error, 1 = warn, 2 = info 3 = debug, 4 = network
  -c TEXT                     Config file to load settings from. Default is config.txt
  -i INT                      Set mining intensity (0 - 64). 0 = auto. Higher means more gpu spends more time hashing. Default is 0.
  --i1 INT                    Set mining intensity (0 - 64). 0 = auto. Higher means more gpu spends more time hashing. Default is 0.
  --i2 INT                    Set mining intensity for second algo (dual mine) (0 - 64). 0 = auto. Higher means more gpu spends more time hashing. Default is 0.
  -g INT                      Ramp up miner rather than start at full speed.
  -b INT                      Cooldown period. 0 = disabled. Higher value means longer time between cooldown periods. default is 0
  --nc INT                    Do not save to the config file (but still read from it).
  --update_frequency INT      Output frequency in milliseconds. Default is 15000.
  --avg_hr_ms INT             Hashrate averaging window. Longer is more stable hashrate reporting. Default is 30000.
  --cpu_validate INT          Validate solutions on cpu before sending to pool.
  --test INT                  Test mine. Useful for setting up overclocks.
  --http_enabled INT          Enable or disable HTTP API. 0 = disabled, 1 = enabled Default is enabled.
  --http_address TEXT         Set IP address for HTTP API to listen on. Default is 0.0.0.0.
  --http_port INT             Set which port the HTTP API listens on. default is 4014.
  --http_password TEXT        Set password for HTTP API. If not set, HTTP API will not be enabled. default is empty.
  --force_opencl INT          Force all devices to use the OpenCL implementation (if possible).
  --reset_oc_dag_gen INT      Reset overclocks before dag generation. Clocks will be set back after dag is generated. 1 = enabled, 0 = disabled
  --devices                   Only log devices. Does not start miner
  --no_watchdog               Do not start watchdog service.
  --disable TEXT ...          Disable specific GPUs from mining, separate by a space. Use device id in the format of pci_bus:pci_device (eg. --disable 1:0 3:0). use --devices to find device id.
  --clear_log_file INT        If 1 (default 0), BzMiner will overwrite the log file on start
  --advanced_config INT       If 1 (default 0), advanced config options will be showin in config.txt.
  --start_script TEXT         If specified, this script will run when BzMiner starts.
  --hung_gpu_ms INT           When GPU does not respond for this amount of time (ms), will be considered hung.
  --crash_script TEXT         When hung GPU is detected, this script will run.
  --hung_gpu_reboot INT       If 1 (default 0), BzMiner will reboot the rig when a hung GPU is detected.
  --hung_gpu_restart_bzminer INT
                              If 1 (default 1), and watchdog is enabled, watchdog will restart BzMiner process when hung GPU is detected.
  --restart_miner_minutes INT If specified and greater than 0, BzMiner watchdog will restart BzMiner process after this amount of time (minutes).
  --reboot_minutes INT        If specified and greater than 0, BzMiner will reboot the rig after this amount of time (minutes).
  --no_color INT              If 1 (default 0), output in console will not have color.
  --log_solutions INT         If 1 (default 1), Solutions will be logged in output (as green).
  --log_date INT              If 1 (default 0), the current date/time will be logged at the start of every line of output.
  --multi_mine_ms INT ...     Time (ms) to mine each algo when dual mining.
  --multi_mine_type INT ...   Multi mine type. 0 = parallel, 1 = alternating (can oc per algo), 2 = mine only during DAG generation. default = 0
  --oc_delay_ms INT           Time (ms) to delay algo switch before/after oc changed for algo.
  --oc_fan_speed INT ...      Set the target fan speed (as percentage) for devices, separated by a space. 0 = auto, -1 = ignore, 100 = max.
  --oc_power_limit INT ...    Set the power limite for devices (in watts), separated by a space. 0 = ignore.
  --oc_core_clock_offset INT ...
                              Set the target core clock offset (in mhz) for devices, separated by a space. 0 = ignore. Will be ignored if oc_lock_core_clock is not 0.
  --oc_memory_clock_offset INT ...
                              Set the target memory clock offset (in mhz) for devices, separated by a space. 0 = ignore. Will be ignored if oc_lock_memory_clock is not 0.
  --oc_lock_core_clock INT ...
                              Lock the core clock for devices (in mhz), separated by a space. Overrides oc_core_clock.
  --oc_lock_memory_clock INT ...
                              Lock the memory clock for devices (in mhz), separated by a space. Overrides oc_memory_clock.
  --oc_unlock_clocks          Unlock the core and memory clocks. Will not mine (same as --devices argument).
  ```
  
  ![image](https://user-images.githubusercontent.com/83083846/147267767-29a8f092-694f-40f0-acb9-bb01fceba41d.png)


## The Configuration File (config.txt)
The configuration file is in the json file format, and can be manually updated, or updated through `bzminer.exe`'s remote commands.

There are three main sections:
- pool_configs - This contains a list of your pools. You can add as many pools as you want, even if your not currently mining to them
- global settings/device defaults - this contains global settings and default settings for devices if the devices do not specify those settings
- device_overrides - this contains a list of the devices on your system. By default they will use the device default settings

BzMiner reads and saves to the configuration file. Upon first running BzMiner, the configuration file will be updated with a new property called `device_overrides`, which contain a list of all the mining devices found on the pc. This is where individual device settings can be set.

```
{
    "pool_configs": [{
            "algorithm": "ethash", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false // if true only lhr cards will mine to this pool
            
            "blockchain_fee": true // whether to enable the blockchain dev team fee (enabled by default, currently only available for kaspa)
        },
        {
            "algorithm": "kawpow", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false // if true only lhr cards will mine to this pool
            
            "blockchain_fee": true // whether to enable the blockchain dev team fee (enabled by default, currently only available for kaspa)
        }],
        
    "pool": [0,1], // one or more pools to mine to. devices that do not specify will mine to these pools (these are indices into the pool_config list)
    
    "rig_name": "rig", // default name of rig, if pool does not specify, will use this as the username/worker name
    
    "log_file": "", // if not empty, the file logs should be written to
    
    "clear_log_file": false, // by default, the log file will rotate (new file each time bzminer starts). set this to true to clear the log file when bzminer starts
    
    "nvidia_only": false, // only mine using nvidia cards
    
    "amd_only": false, // only mine using amd cards
    
    "lock_config": false, // if true, bzminer will never write to this file
    
    "extra_dev_fee": 0.0, // additional percentage to add to dev fee (thanks for your support!)
    
    "advanced_config": false, // show advanced config options (after setting true, must run bzminer once so it can update this file)
    
    "advanced_display_config": false, // show advanced display options (after setting true, must run bzminer once so it can update this file)
    
    "log_solutions": true, // whether to log each solution found in the output
    
    "x_display": ":0", // which x display to use (for nvidia, linux only)
    
    "start_x": false, // attempt to start an x display (for nvidia, linux only)
    
    "device_overrides": [ // list of devices and their specific settings
        {
            "uid": "1:0", // device id (pci_bus_id:pci_device_id)
            
            "name": "EVGA RTX 3090 FTW3 Ultra", // device display name
            
            "start_mining": true // set to false to disable this device from mining
        }
    ]
}
```

With both "advanced_config" and "advanced_display_config" turned on, the full config file is as follows:

```
{
    "pool_configs": [{
            "algorithm": "ethash", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false, // if true only lhr cards will mine to this pool
            
            "delay_before_connection_retry": 3000, // milliseconds between each reconnection retry (minimum is 1 second)
            
            "no_work_timeout": 30000, // ms after connection begins before no work timeout is triggered and reconnection happens
            
            "test": false, // run this algo in test mode (will not connect to a pool)
            
            "test_iteration_ms": 15000 //  seconds between new work in test mode
            
            "blockchain_fee": true // whether to enable the blockchain dev team fee (enabled by default, currently only available for kaspa)
        },
        {
            "algorithm": "kawpow", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false, // if true only lhr cards will mine to this pool
            
            "delay_before_connection_retry": 3000, // milliseconds between each reconnection retry (minimum is 1 second)
            
            "no_work_timeout": 30000, // ms after connection begins before no work timeout is triggered and reconnection happens
            
            "test": false, // run this algo in test mode (will not connect to a pool)
            
            "test_iteration_ms": 15000 //  seconds between new work in test mode
            
            "blockchain_fee": true // whether to enable the blockchain dev team fee (enabled by default, currently only available for kaspa)
        }],

    "pool": [0,1], // one or more pools to mine to. devices that do not specify will mine to these pools (these are indices into the pool_config list)

    "intensity": [0], // set mining intensity for device. 0-64, 0 = auto

    "repair_dag": [true], // Whether the dag (currently only ethash and etchash) should be repaired if an issue occurs. Useful for extreme overclocking. this has an impact on dag generation time
    
    "calc_dag_crc": [false], // Whether to calculate the dag crc. this will have an impact on dag generation time.
    
    "ramp_up": [false], // Starts mining at a slower speed and slowly ramps up to full mining speed
    
    "cooldown_period": [0], // Allows the GPU to cool down after some time. 0 = off, default = 0, higher values means longer intervals between cooldowns
    
    "cpu_validate": [false], // Whether solutions from this device should be validated on the CPU before sent to pool
    
    "reset_oc_dag_gen": [false], // If true, will reset overclocks before dag is generated. after dag is generated, will set overclocks back to what they currently are. Useful for very high OC
    
    "multi_mine_ms": [5000, 15000], // how long each algo should mine when dual mining
    
    "multi_mine_type": 1, // 0 = parallel mine, 1 = alternate mine (can oc per algo), 2 = DAG gen only mining
    
    "temp_start": 80, // temperature (C) that device should start mining at after it has stopped due to temp_stop
    
    "temp_stop": 120, // temperature (C) that device should stop mining at
    
    "rig_name": "rig", // default name of rig, if pool does not specify, will use this as the username/worker name
    
    "log_file": "", // if not empty, the file logs should be written to
    
    "clear_log_file": false, // by default, the log file will not be cleared when bzminer starts. set this to true to clear the log file when bzminer starts
    
    "nvidia_only": false, // only mine using nvidia cards
    
    "amd_only": false, // only mine using amd cards
    
    "lock_config": false, // if true, bzminer will never write to this file
    
    "extra_dev_fee": 0.0, // additional percentage to add to dev fee (thanks for your support!)
    
    "advanced_config": true, // show advanced config options (after setting true, must run bzminer once so it can update this file)
    
    "advanced_display_config": true, // show advanced display options (after setting true, must run bzminer once so it can update this file)
    
    "launch_on_boot": false, // if true, will automatically launch bzminer when pc turns on (windows only)
    
    "start_script": "start.bat", // Optional script to run when bzminer starts up
    
    "hung_gpu_ms": 30000, // After GPU is unresponsive for this number of milliseconds, it is considered "hung"
    
    "crash_script": "crash.bat", // Optional script to call when a hung gpu is detected
    
    "hung_gpu_reboot": false, // If true, will reboot rig when a hung gpu is detected
    
    "hung_gpu_restart_bzminer": true, // If true, and watchdog is running, will restart bzminer when a hung gpu is detected
    
    "restart_miner_minutes": 0, // restart bzminer after this amount of time (minutes), IF watchdog is enabled
    
    "reboot_minutes": 0, // reboot pc after this amount of time (minutes)
    
    "no_color": false, // disable all color output
    
    "verbosity": 2, // log level (0 = errors only, 1 = warnings, 2 = info (default), 3 = debug, 4 = network, 5 = trace (not available in release)
    
    "update_frequency": 15000, // how often to log info (in milliseconds)
    
    "avg_hr_ms": 30000, // time to average hashrate (ms). Higher time means more stable hashrate, lower time means more realtime
    
    "log_solutions": true,  // whether to log each solution found in the output
    
    "log_date": false, // whether to log the date on each log line
    
    "http_password": "", // password for http api
    
    "http_port": 4014, // port to use for http api
    
    "http_address": "0.0.0.0", // address to listen on for http api
    
    "http_enabled": true, // set to false to disable http api
    
    "disable_udp": false, // If true, the UDP service is disabled and this device will not show up in other instances of the BzMiner GUI
    
    "x_display": ":0", // which x display to use (for nvidia, linux only)
    
    "start_x": false, // attempt to start an x display (for nvidia, linux only)
    
    "display_settings": { // advanced display settings
    
        "table_width": 100, // number of characters wide the tables are. 0 = auto
        
        "wrap": true, // whether the table should allow columns to wrap (set to false to disable cells from wrapping their contents to the next line)
        
        "title_align": 1, // table title horrizontal alignment. 0 = left, 1 = center, 2 = right. default is 1 = center
        
        "date_align": 0, // table date alignment. 0 = left, 1 = center, 2 = right. default is 0 = left
        
        "hide_unused_devices": false // if true, will hide any devices that are not mining (with start_mining: false setting)
        
        "show_devices": true, // show the devices table
        
        "devices_title_color": 7, // set the color of the devices table title
        
        "devices_border_color": 8, // set the color of the devices table border
        
        "devices_columns": "#,name:nw,free,total,core,mem,fan,pwr,temp", // which columns to show for devices table. supported list below
        
        "show_pool": true, // whether to show the pool tables
        
        "pool_title_color": 7, // pool table title color
        
        "pool_border_color": 8, // pool table border color
        
        "pool_columns": "#,uptime,a/r/i,avg,eff,pool mh,miner mh,status" // which columns to show in the pool tables
    },
    
    "oc_delay_ms": 50, // for dual mining only, time before/after oc is applied when switching to next algo
    "oc_fan_speed": [], // List (optionally of lists for dual mining oc's) of fan speeds (%)
    "oc_power_limit": [], // List (optionally of lists for dual mining oc's) of power limits (watts)
    "oc_core_clock_offset": [], // List (optionally of lists for dual mining oc's) of core offsets (mhz)
    "oc_memory_clock_offset": [], // List (optionally of lists for dual mining oc's) of memory offsets (mhz)
    "oc_lock_core_clock": [], // List (optionally of lists for dual mining oc's) of locked core clocks (mhz, overrides core offset when not 0)
    "oc_lock_memory_clock": [], // List (optionally of lists for dual mining oc's) of locked memory clocks (mhz, overrides memory offset when not 0)
    
    "device_overrides": [{
            // device info/settings
            "uid": "1:0", // pci bus id : pci device id. this uniquely identifies the device as long as it doesn't change pci slots
            
            "name": "EVGA RTX 3090 FTW3 Ultra", // name of the device
            
            "start_mining": true, // if false, device will not mine
            
            "lhr": false, // whether this device is an lhr card or not ( set automatically )
            
            "temp_start": 80, // temperature (C) that device should start mining at after it has stopped due to temp_stop
            
            "temp_stop": 120, // temperature (C) that device should stop mining at

            // pool settings for this device
            "pool": [0,1], // list of pools this device should mine to
            
            "intensity": [0], // mining intensity. 0-64, 0 = auto. higher values generally means higher hashrate, but higher chance of stales
            
            "repair_dag": [true],  // Whether the dag (currently only ethash and etchash) should be repaired if an issue occurs. Useful for extreme overclocking. this has an impact on dag generation time
            
            "calc_dag_crc": [false], // Whether to calculate the dag crc. this will have an impact on dag generation time.
            
            "ramp_up": [false], // Starts mining at a slower speed and slowly ramps up to full mining speed
            
            "cooldown_period": [0], // Allows the GPU to cool down after some time. 0 = off, default = 0, higher values means longer intervals between cooldowns
            
            "reset_oc_dag_gen": [false], // If true, will reset overclocks before dag is generated. after dag is generated, will set overclocks back to what they currently are. Useful for very high OC
            
            "force_opencl": [false], // force device to mine on OpenCL (if available, only affects nvidia)
            
            "dag_dev_ref": [""], // Instead of this device generating the dag, it can copy the dag from another device. Nvidia only currently. useful for high oc's
 
            // overclocking
            "oc_fan_speed": [-1], // set the devices fan speed. -1 = do not change, 0 = auto, >0 = lock at percentage. Optionally a list for dual mining
            
            "oc_power_limit": [0], // set the max power consumption (watts) the device will use. 0 = do not change, >0 = max watts. Optionally a list for dual mining
            
            "oc_core_clock_offset": [0], // set the core clock speed offset in MHz. 0 = do not change, !0 = set clock speed offset. Optionally a list for dual mining
            
            "oc_memory_clock_offset": [0], // set the memory clock speed offset in MHz. 0 = do not change, !0 = set memory speed offset. Optionally a list for dual mining
            
            "oc_lock_core_clock": [0], // Lock the core clock to a specific mhz, if device allows it. Optionally a list for dual mining
            
            "oc_lock_memory_clock": [0] // Lock the memory clock to a specific mhz, if device allows it. Optionally a list for dual mining
        }]
}
```

### How to disable specific GPUs

GPU's can be disabled either through the command line or through `config.txt`

The command line argument is `--disable` and expects a space separated list of gpu unique ids, or indexes

eg.

`bzminer --disable 1:0 3:0`

or 

`bzminer --disable 0 3 5`

Optionally they can be disabled directly from the `config.txt` file, under the gpu in the config.txt file, set `auto_start` to false

### LHR Strategy (Dual Mining) EXPERIMENTAL!
The current strategy BzMiner has to deal with LHR cards, is dual/multi coin mining, currently using ethash and kawpow

To employ this strategy:
- Add both an ethash pool and a kawpow (or other non memory hard algo) pool to the config file
- Optionally set the kawpow pool setting "lhr_only" to true
- Set either the global "pool" setting, or device override "pool" setting to use both the ethash and kawpow pool. eg. "pool": [0, 1]
- Optionally set the device override "lhr" setting to true, otherwise bzminer will attempt to determine whether the card is lhr or not. eg. "lhr": true
- Run bzminer using that config. eg. `bzminer -c config.txt`

## Overclock per Dual Mine Algo
BzMiner allows you to set an overclock for each algo you are dual mining. Be aware that extreme oc's can much easier cause a GPU to hang (crash) in this scenario, and for that reason the config option `oc_delay_ms` is used, to give the oc time to kick in before switching to next algo.

Here's a sample dual mine config with oc's per algo:

```
{
    "pool_configs": [{
            "algorithm": "ethash",
            "wallet": "0000",
            "url": ["stratum+tcp://us2.ethermine.org:4444"],
            "lhr_only": false
        }, {
            "algorithm": "alph",
            "wallet": "0000",
            "url": ["stratum+tcp://eu.metapool.tech:20032", "stratum+tcp://pool.woolypooly.com:3106"],
            "username": "alph_rig",
            "lhr_only": false
        }],
    "pool": [0, 1],
    "intensity": [0],
    "multi_mine_ms": [5000, 15000],
    "rig_name": "ethash_rig",
    "log_file": "",
    "clear_log_file": false,
    "nvidia_only": false,
    "amd_only": false,
    "lock_config": false,
    "advanced_config": false,
    "oc_delay_ms": 100,
    "device_overrides": [{
            "uid": "1:0",
            "name": "EVGA RTX 3090 FTW3 Ultra",
            "start_mining": true,
            "lhr_mine_ms": [5000, 15000],
            "oc_fan_speed": [-1],
            "oc_power_limit": [0],
            "oc_core_clock_offset": [-200, 50],
            "oc_memory_clock_offset": [1100, 0],
            "oc_lock_core_clock": [0],
            "oc_lock_memory_clock": [0, 810]
        }]
}
```

## Display configuration

BzMiner allows you to customize the output displays. This includes the table width, title color, border color, and which columns in each table to show.

There are two tables that are logged by default, the devices table, and the pool tables. On each display update, the devices table is logged, which contain information about all the devices on the system. This table is not in the context of a pool, so does not support some of the column options the pool tables support, including hashrate.

The Pool table is logged for each pool that devices are mining to. If there are two devices on the system, and each device is mining to a different pool (or different algorithm), then there will be two pool tables in the output, one for each pool. This table supports all the columns in the device table, as well as columns specific to the pool context.

default device column configuration: `"#,name:nw,cfg,free,total,core,mem,fan,pwr,temp"`

default pool column configuration: `"#,uptime,a/r/i,avg,eff,pool mh,miner mh,status:nw"`

### Supported Device table columns
- `#` - The device id in the format of `pci_bus_id:pci_device_id`
- `name` - The display name of the device
- `free` - Free device memory
- `total` - Total device memory
- `core` - Current core clock speed
- `mem` - Current memory clock speed
- `fan` - Fan speed
- `pwr` - Power Usage
- `temp` - Temperature. In the format of `core temp/memory temp`
- `status` - Device status for all pools

### Supported Pool table columns (In addition to all device table supported columns)
- `uptime` - The time the device has been mining without stopping or resetting
- `cfg` - The device configuration. eg. i6 = intensity 6
- `a/r/i` - Accepted/Rejected/Invalid shares
- `tbs` - Average time between shares
- `eff` - Efficiency, in the units of hashes per watt
- `pool mh` - Estimated effective hashrate based on pool difficulty and shares found (effective/current hashrate on some pools)
- `miner mh` - Current miner speed (reported hashrate on some pools)
- `status` - Device/Pool status.

The pool table has an additional row at the bottom which is the summary of all devices mining to that pool. The status column for the pool includes the epoch number, difficulty, and average latency to pool.

### Customizing the display
- Colors are integers and correspond to normal terminal color codes. Default BzMiner logging color is 7.
- Columns are a comma separated string. do not include spaces unless the column name includes spaces (eg. "pool mh"). Columns can have options (currently just one), which is specified by adding a colon (:) after the column name, and specifying the option, eg `name:nw`. Two supported options; `nw`, which prevents that particular column from wrapping to the next line if the column cannot get wide enough to support the entire value on one line, and `nc`, which disables coloring of the column values

## Pool URLs

Urls should be in the form of `{protocol}+{tcp/ssl}:{url}:{port}` (username and password are optional)

eg. `stratum+tcp://us1.ethermine.org:4444`

If username and password are required, eg. `{protocol}+{tcp/ssl}:<{username}:{password}@>{url}:{port}`, they can be set per device in the config file, or through the command line. If device pool_username is blank, it will use the rig name from the config file. if pool_password is blank, it will use the default pool_password for the rig in the config file. Leave both pool_password blank if no password is required for the pool. Do not put the username and password in the url (If there are enough requests for this, this can be added in a future release)

BzMiner supports 4 network protocols
- Node - use `node` or `solo`
- Stratum - use `stratum`
- Eth Proxy - use `ethproxy`
- Ethereum Stratum v1.0.0 - use `ethstratum`
- Ethereum Stratum v2.0.0 - use `ethstratum2`
- Alephium Binary Stratum - use `alphstratum`
- HTTP or HTTPS GetWork - use `http` or `https`

BzMiner will attempt to auto select a protocol the pool supports if the provided protocol does not succeed in establishing a connection with the pool.


## HTTP API

HTTP API can be enabled by launching bzminer with the --http_enabled flag, or by setting "http_enabled": true in the config file.

For more details, enable the http api, and go to `http://{http_address}:{http port}/help`. If you are on the same computer, you can use `localhost` or `127.0.0.1` as the http_address.

A simple viewer can be found at `/index` when the HTTP API is enabled. `index.html` is provided for your convenience.

## Watchdog service

By default, the wathdog service is running. The watdog service watches for:

- Hung GPU (GPU not responding after a period of time)
- Unresponsive network activity
- Unresponsive main loop
- BzMiner crashing

If a hung GPU is detected, the watchdog service will optionally call a script specified in the config file, then optionally reboot the rig if specified in the config file.

The last log (and output) from BzMiner before the optional script is called is in the format of:

`<Error> Hung GPU detected! (bus_id:device_id)`

Which could be read from the called script if needed.

The watchdog also restarts BzMiner if BzMiner crashes for whatever reason. This shouldn't normally happen, but the occasional bug can cause BzMiner to crash, and with the wathdog service, it'll start back up again. The watchdog service also restarts BzMiner in instances of unresponsive network activity and an unresponsive main loop.
