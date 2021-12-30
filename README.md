## Why use BzMiner (v5.1)?
- Supported Algos:
    - Ethash (AMD, Nvidia)
    - Etchash (AMD, Nvidia) 
    - Kawpow (AMD, Nvidia)
    - Olhash (AMD, Nvidia, 1% dev fee)
- Low dev fee of 0.5%
- (experimental) LHR Strategy dual/multi coin mining (ethash/kawpow)
- Control algorithm/pool/wallet mining PER device, as well as Multi-coin mining on a single GPU
- Awesome Easy to use Linux and Windows miner (GUI available through browser)
- Hung GPU detection
- Watchdog service which automatically restarts BzMiner if anything goes wrong (such as a hung gpu)
- Customizable display
- Memory junction temperature monitoring (windows only)
- HTTP API!
- Remote management of all mining rigs on your network
- Fast Cuda v11 and OpenCL Ethereum/ethash and Ethereum classic miner
- Constantly being improved based on feedback from you!
- Ability to OC in miner (OC's are removed during DAG generation to prevent invalid DAG generation)
- Advanced mining features, including cooldown and ramp up 
- DAG Validation for very high OC cards
- AMD ethash and etchash mining (still experimental)
- Auto intensity, dyanamically adjusts gpu workloads, reducing stales while keeping hashrate high

## NOTE: Requires latest NVIDIA Drivers


## Mining OS's that include BzMiner
- MMPOS - https://app.mmpos.eu/
- HiveOS - https://hiveos.farm/


## Current planned major features by version (not including minor releases)
- v7.0 next algo: autolykos (ERG/Ergo)
- v8.0 next algo: firopow (FIRO)


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
bzminer -a etchash -w 0x0000000000000000000000000000000000000000 -p stratum+tcp://us1-etc.ethermine.org:4444 stratum+tcp://eu1-etc.ethermine.org:4444 -r worker_name
```

### Ethereum Classic

```
bzminer -a ethash -w 0x0000000000000000000000000000000000000000 -p stratum+tcp://usw-eth.hiveon.net:4444 stratum+tcp://us1.ethermine.org:4444 -r worker_name
```

### Ravencoin

```
bzminer -a kawpow -w 0x0000000000000000000000000000000000000000 -p stratum+ssl://stratum-ravencoin.flypool.org:3443 -r worker_name
```

### Overline

```
bzminer -a olhash -w 0x0000000000000000000000000000000000000000 -p ethproxy+ssl://us.extremehash.net:3443 ethproxy+ssl://eu-ol.extremehash.net:3443 -r worker_name
```

## GUI
The GUI desktop app has been discontinued as of v4.3 in favor of the browser gui, which can be accessed by opening `index.html`, or by navigating to `http://your-rigs-ip:port/` in your favorite browser.

The GUI will display (and allow you to navigate to) other rigs running BzMiner. This is done through a UDP broadcast message which can be disabled on instances of BzMiner by setting "disable_udp" to true in the config file. 

![image](https://user-images.githubusercontent.com/83083846/147267304-8357dd99-1638-4d83-a41a-0ebabd01cd4b.png)


## CLI
BzMiner is a command line interface. Simply update `config.txt` and launch `bzminer`.

`bzminer` has an optional parameter, `-c`, which can be called to load a different configuration file. eg. `bzminer -c custom_config.txt`.

`bzminer` also has optional parameters for setting the pool url, wallet address, algorithm and rig/worker name for all devices

```
>bzminer -h
BZMiner is an enhanced CUDA Ethereum Ethash miner
Usage: bzminer [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -a TEXT                     Default Mining algorithm. eg. 'ethash'
  -r TEXT                     Default Rig (worker/username) name. eg. 'Rig'
  --pool_password TEXT        Default Pool password
  --nvidia INT                Only mine with Nvidia devices (0 = false, 1 = true)
  --amd INT                   Only mine with AMD devices (0 = false, 1 = true)
  -o TEXT                     If provided, output will be logged to this file
  -w TEXT                     Wallet Address
  -p TEXT ...                 Array of Pool Addresses. eg. stratum+tcp://us1.ethermine.org:4444  stratum+tcp://us2.ethermine.org:4444
  -v INT                      Set log verbosity. 0 = Error, 1 = warn, 2 = info 3 = debug, 4 = network
  -c TEXT                     Config file to load settings from. Default is config.txt
  -i INT                      Set mining intensity (1 - 32). Higher means more gpu spends more time hashing. Default is 8.
  -u INT                      Update frequency in milliseconds. Default is 10000.
  -t INT                      Thrashing. 0 = off (default), 1 = on. Turns stales to valid solutions at cost of high CPU usage. less stales
  -s INT                      Stales are ok. 1 = OK, 0 = Not ok. default 0. If OK, spend more time mining, get more valid shares. more stales
  -g INT                      Ramp up miner rather than start at full speed.
  -b INT                      Cooldown period. 0 = disabled. Higher value means longer time between cooldown periods. default is 0
  --nc INT                    Do not save to the config file (but still read from it).
  --cpu_validate INT          Validate solutions on cpu before sending to pool.
  --test INT                  Test mine. Useful for setting up overclocks.
  --http_enabled INT          Enable or disable HTTP API. 0 = disabled, 1 = enabled Default is enabled.
  --http_address TEXT         Set IP address for HTTP API to listen on. Default is 0.0.0.0.
  --http_port INT             Set which port the HTTP API listens on. default is 4014.
  --http_password TEXT        Set password for HTTP API. If not set, HTTP API will not be enabled. default is empty.
  --force_opencl INT          Force all devices to use the OpenCL implementation (if possible).
  --reset_oc_dag_gen INT      Reset overclocks before dag generation. Clocks will be set back after dag is generated. 1 = enabled, 0 = disabled
  --no_watchdog               Do not start watchdog service
  --devices                   Only log devices. Does not start miner
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
        },
        {
            "algorithm": "kawpow", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false // if true only lhr cards will mine to this pool
        }],
        
    "pool": [0,1], // one or more pools to mine to. devices that do not specify will mine to these pools (these are indices into the pool_config list)
    
    "rig_name": "rig", // default name of rig, if pool does not specify, will use this as the username/worker name
    
    "log_file": "", // if not empty, the file logs should be written to
    
    "nvidia_only": false, // only mine using nvidia cards
    
    "amd_only": false, // only mine using amd cards
    
    "auto_detect_lhr": true, // auto detect if a card has lhr (otherwise it can be manually specified in the device_override)
    
    "lock_config": false, // if true, bzminer will never write to this file
    
    "advanced_config": false, // show advanced config options (after setting true, must run bzminer once so it can update this file)
    
    "advanced_display_config": false, // show advanced display options (after setting true, must run bzminer once so it can update this file)
    
    "log_solutions": true, // whether to log each solution found in the output
    
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
            
            "lhr_only": false // if true only lhr cards will mine to this pool
        },
        {
            "algorithm": "kawpow", // pool algorithm
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet to mine to
            
            "url": ["stratum+tcp://yourpool:4444", "stratum+tcp://yourpool:4444"], // list of urls to connect to. will rotate through urls if they fail
            
            "username": "rig", // username/worker name to use
            
            "lhr_only": false // if true only lhr cards will mine to this pool
        }],

    "pool": [0,1], // one or more pools to mine to. devices that do not specify will mine to these pools (these are indices into the pool_config list)

    "intensity": [0], // set mining intensity for device. 0-64, 0 = auto

    "repair_dag": [true], // Whether the dag (currently only ethash and etchash) should be repaired if an issue occurs. Useful for extreme overclocking. this has an impact on dag generation time
    
    "calc_dag_crc": [false], // Whether to calculate the dag crc. this will have an impact on dag generation time.
    
    "ramp_up": [false], // Starts mining at a slower speed and slowly ramps up to full mining speed
    
    "cooldown_period": [0], // Allows the GPU to cool down after some time. 0 = off, default = 0, higher values means longer intervals between cooldowns
    
    "cpu_validate": [false], // Whether solutions from this device should be validated on the CPU before sent to pool
    
    "reset_oc_dag_gen": [false], // If true, will reset overclocks before dag is generated. after dag is generated, will set overclocks back to what they currently are. Useful for very high OC
    
    "temp_start": 80, // temperature (C) that device should start mining at after it has stopped due to temp_stop
    
    "temp_stop": 120, // temperature (C) that device should stop mining at
    
    "rig_name": "rig", // default name of rig, if pool does not specify, will use this as the username/worker name
    
    "log_file": "", // if not empty, the file logs should be written to
    
    "nvidia_only": false, // only mine using nvidia cards
    
    "amd_only": false, // only mine using amd cards
    
    "auto_detect_lhr": true, // auto detect if a card has lhr (otherwise it can be manually specified in the device_override)
    
    "lock_config": false, // if true, bzminer will never write to this file
    
    "advanced_config": true, // show advanced config options (after setting true, must run bzminer once so it can update this file)
    
    "advanced_display_config": true, // show advanced display options (after setting true, must run bzminer once so it can update this file)
    
    "launch_on_boot": false, // if true, will automatically launch bzminer when pc turns on (windows only)
    
    "hung_gpu_ms": 30000, // After GPU is unresponsive for this number of milliseconds, it is considered "hung"
    
    "crash_script": "", // When a hung gpu is detected, call this script
    
    "hung_gpu_reboot": false, // If true, will reboot rig when a hung gpu is detected
    
    "hung_gpu_restart_bzminer": true, // If true, and watchdog is running, will restart bzminer when a hung gpu is detected
    
    "no_color": false, // disable all color output
    
    "verbosity": 2, // log level (0 = errors only, 1 = warnings, 2 = info (default), 3 = debug, 4 = network, 5 = trace (not available in release)
    
    "update_frequency": 15000, // how often to log info (in milliseconds)
    
    "log_solutions": true,  // whether to log each solution found in the output
    
    "log_date": false, // whether to log the date on each log line
    
    "http_password": "", // password for http api
    
    "http_port": 4014, // port to use for http api
    
    "http_address": "0.0.0.0", // address to listen on for http api
    
    "http_enabled": true, // set to false to disable http api
    
    "disable_udp": false, // If true, the UDP service is disabled and this device will not show up in other instances of the BzMiner GUI
    
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
    "device_overrides": [{
            // device info/settings
            "uid": "1:0", // pci bus id : pci device id. this uniquely identifies the device as long as it doesn't change pci slots
            
            "name": "EVGA RTX 3090 FTW3 Ultra", // name of the device
            
            "start_mining": true, // if false, device will not mine
            
            "lhr": false, // whether this device is an lhr card or not. if false, bzminer will attempt to auto detect
            
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
            "oc_fan_speed": -1, // set the devices fan speed. -1 = do not change, 0 = auto, >0 = lock at percentage
            
            "oc_power_limit": 0, // set the max power consumption (watts) the device will use. 0 = do not change, >0 = max watts
            
            "oc_core_clock": 0, // set the core clock speed offset in MHz. 0 = do not change, !0 = set clock speed offset
            
            "oc_memory_clock": 0 // set the memory clock speed offset in MHz. 0 = do not change, !0 = set memory speed offset
        }]
}
```

### (experimental) LHR Strategy
The current strategy BzMiner has to deal with LHR cards, is dual/multi coin mining, currently using ethash and kawpow

To employ this strategy:
- Add both an ethash pool and a kawpow pool to the config file
- Optionally set the kawpow pool setting "lhr_only" to true
- Set either the global "pool" setting, or device override "pool" setting to use both the ethash and kawpow pool. eg. "pool": [0, 1]
- Optionally set the device override "lhr" setting to true, otherwise bzminer will attempt to determine whether the card is lhr or not. eg. "lhr": true
- Run bzminer using that config. eg. `bzminer -c config.txt`

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
- `avg` - Average time between each share
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
- Stratum - use `stratum`
- Eth Proxy - use `ethproxy`
- Ethereum Stratum v1.0.0 - use `ethstratum`
- Ethereum Stratum v2.0.0 - use `ethstratum2`

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
