## Why use BzMiner (v4.7)?
- Low dev fee of 0.5%
- Customizable display
- Now monitoring memory junction temperature (windows only)
- Awesome Easy to use Linux and Windows miner (GUI available through browser)
- HTTP API!
- Remote management of all mining rigs on your network
- Fastest Cuda v11 and OpenCL Ethereum/ethash and Ethereum classic miner
- Control algorithm/pool/wallet mining PER device!
- Constantly being improved based on feedback from you!
- Ability to OC in miner (OC's are removed during DAG generation to prevent invalid DAG generation)
- Advanced mining features, including cooldown periods, thrashing, stales_ok and ramp up 
- DAG Validation for very high OC cards
- AMD ethash and etchash mining (still experimental)
- Auto intensity, dyanamically adjusts gpu workloads, reducing stales while keeping hashrate high
- Supported Algos:
    - Ethash (AMD, Nvidia)
    - Etchash (AMD, Nvidia) 


## Mining OS's that include BzMiner
- MMPOS - https://app.mmpos.eu/
- HiveOS - https://hiveos.farm/


## Current planned major features by version (not including minor releases)
- v5.0 next algo: KawPow (RVN/Ravencoin), autolykos (ERG/Ergo), undecided as of now


## Discord Server
https://discord.gg/NRty3PCVdB

## Official Website
http://www.bzminer.com


## Requirements
- At least one Nvidia GPU and Nvidia drivers
- Or at least one AMD GPU and AMD drivers

## Quick Start
Launch `bzminer` with the wallet and pool address as parameters:

`bzminer -w 0x0000000000000000000000000000000000000000 -p ethstratum+tcp://eth.geodns.flexpool.io:4444 -r worker_name`

You may provide more than one pool address to -p for fallback pools on network errors:
`bzminer -w 0x0000000000000000000000000000000000000000 -p ethstratum+tcp://eth.geodns.flexpool.io:4444 stratum+tcp://usw-eth.hiveon.net:4444 -r worker_name`

Optionally you can edit `config.txt` and launch `bzminer`. You can specify the config file with the `-c` argument:

`bzminer -c eth-config.txt`


## GUI (bzminergui.exe)
The GUI desktop app has been discontinued as of v4.3 in favor of the browser gui, which can be accessed by opening `index.html`, or by navigating to `http://your-rigs-ip:port/` in your favorite browser.

![image](https://user-images.githubusercontent.com/83083846/136844596-db8331a0-ab7e-42ee-8315-49e7f2deea0a.png)


## CLI (bzminer.exe)
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
  --devices                   Only log devices. Does not start miner
  ```
  ![image](https://user-images.githubusercontent.com/83083846/138520340-10443ec7-f1a1-439c-90ef-31712e8881a3.png)


## The Configuration File (config.txt)
The configuration file is in the json file format, and can be manually updated, or updated through `bzminer.exe`'s remote commands.

BzMiner reads and saves to the configuration file. Upon first running BzMiner, the configuration file will be updated with a new property called `device_overrides`, which contain a list of all the mining devices found on the pc. This is where individual device settings can be set.

```
{
    "algorithm": "ethash", // default algorithm new devices will use
    "wallet": "0x0000000000000000000000000000000000000000", // default wallet/username new devices will use
    "pool": [  // default pool address new devices will use. This can be an array or a single string
        "stratum+tcp://yourpool:4444",
        "stratum+tcp://yourpool:4444"
    ],
    "rig": "rig", // name of this rig. A.K.A. worker/username
    "pool_password": "", // password to use for pool. Leave blank if no password required
    "nvidia_only": false, // Mine only with Nvidia devices
    "amd_only": false, // Mine only with AMD devices
    "log_file": "", // file to output logs to. leave blank to disable logging to a file
    "advanced_config": false, // set this to true to show additional options in the config file
    "advanced_display_config": false, // shows options for configuring display
    "device_overrides": [ // list of individual device settings
        {
            "uid": "1:0", // device id (pci_bus_id:pci_device_id)
            "name": "EVGA RTX 3090 FTW3 Ultra", // device display name
            "start_mining": true // set to false to disable this device from mining
        }
    ]
}
```

With both "advanced_config" and "advanced_display_config" turned out, the full config file is as follows:

```
{
    "algorithm": "ethash", // default algorithm new devices will use
    
    "wallet": "0x0000000000000000000000000000000000000000", // default wallet/username new devices will use
    
    "pool": [  // default pool address new devices will use. This can be an array or a single string
        "stratum+tcp://yourpool:4444",
        "stratum+tcp://yourpool:4444"
    ],
    
    "rig": "rig", // name of this rig. A.K.A. default worker/username
    
    "pool_password": "", // password to use for pool. Leave blank if no password required
    
    "nvidia_only": false, // Mine only with Nvidia devices
    
    "amd_only": false, // Mine only with AMD devices
    
    "log_file": "", // file to output logs to. leave blank to disable logging to a file
    
    "launch_cli_on_start": false, // whether cli.exe should start on reboot (windows only)
    
    "launch_gui_on_start": false, // whether bzminer.exe should start on reboot. If both are true, only bzminer.exe will start (windows only)
    
    "farm": "farm", // name of the farm this rig belongs to
    
    "verbosity": 2, // log verbosity. 0 = Error, 1 = Warn, 2 = Info (default), 3 = debug, 4 = network
    
    "log_date": false, // whether to log the date on each line
    
    "temp_start": 80, // default resume mining once device reaches this temp
    
    "temp_stop": 120, // default pause mining on device if over this temp (120 is basically disable pausing on temp)
    
    "intensity": 8 // default value between 1 and 32. Higher value means more time calculating hashes but higher chance of stales
    
    "update_frequency": 10000, // update output frequency in milliseconds,
    
    "thrash": false, // Turn thrashing on or off. Thrashing utilizes more CPU in an attemp to turn stale solutions into valid solutions
    
    "stales_ok": false, // Some pools pay for stales. If this is true, BzMiner will not attempt to prevent stales, allowing more time to mine.
    
    "ramp_up": false, // If true, GPUs will start mining slowly after DAG generation, then ramp up to full speed.
    
    "cpu_validate": false, // Whether or not solutions should be validated on the CPU before sending to the pool
    
    "http_enabled": true, // Whether the HTTP API should be enabled or not. Default is enabled
    
    "http_address": "0.0.0.0", // IP address HTTP API will listen on. Default is 0.0.0.0 (all ip addresses)
    
    "http_port": 4014, // HTTP API listen port
    
    "http_password": "", // HTTP API password. If this is empty, HTTP API is disabled.
    
    "cooldown_period": 0, // allow the gpu to cooldown after a period of time. default is 0. Higher value means more time between cooldown periods.
    
    "reset_oc_dag_gen": false, // Resets overclocks before DAG is generated. Sets overclocks back after DAG has completely been generated. Useful for high oc when getting a lot of invalid shares due to invalid DAG generation
    
    "advanced_config": true, // set this to true to show additional options in the config file
    
    "advanced_display_config": true, // set to false to disable this device from mining
    
    "display_settings": { // advanced display configuration settings
        "table_width": 100, // number of characters wide the tables are
        "wrap": true, // whether the table should allow columns to wrap (set to false to disable cells from wrapping their contents to the next line)
        "title_align": 1, // table title horrizontal alignment. 0 = left, 1 = center, 2 = right. default is 1 = center
        "date_align": 0, // table date alignment. 0 = left, 1 = center, 2 = right. default is 0 = left
        "hide_unused_devices": false // if true, will hide any devices that are not mining (with start_mining: false setting)
        "show_devices": true, // show the devices table
        "devices_title_color": 7, // set the color of the devices table title
        "devices_border_color": 8, // set the color of the devices table border
        "devices_columns": "#,name:nw,cfg,free,total,core,mem,fan,pwr,temp", // which columns to show for devices table. supported list below
        "show_pool": true, // whether to show the pool tables
        "pool_title_color": 7, // pool table title color
        "pool_border_color": 8, // pool table border color
        "pool_columns": "#,uptime,a/r/i,avg,eff,pool mh,miner mh,status:nw" // which columns to show in the pool tables
    },
    
    "device_overrides": [ // list of individual device settings
        {
            "uid": "37:0", // pci bus id : pci device id. this uniquely identifies the device as long as it doesn't change pci slots
            
            "algorithm": "ethash", // algorithm this device should mine using
            
            "wallet": "0x0000000000000000000000000000000000000000", // wallet/username this device should use
            
            "pool": [  // default pool address new devices will use. This can be an array or a single string
                "stratum+tcp://yourpool:4444",
                "stratum+tcp://yourpool:4444"
            ],
            
            "pool_username": "rig", // worker/username
            
            "pool_password": "", // password for pool if required
            
            "temp_start": 80, // resume mining once device reaches this temp
            
            "temp_stop": 120,// dpause mining on device if over this temp
            
            "start_mining": true // whether to start mining on this device when BzMiner starts
            
            "intensity": 8 // value between 1 and 32. Higher value means more time calculating hashes but higher chance of stales,
            
            "thrash": false, // Turn thrashing on or off. Thrashing utilizes more CPU in an attemp to turn stale solutions into valid solutions
            
            "stales_ok": false, // Some pools pay for stales. If this is true, BzMiner will not attempt to prevent stales, allowing more time to mine.
            
            "test": false, // Run this device in test mode. This is useful for getting overclocks just the way you want.
            
            "test_diff": 1 // test mode difficulty,
            
            "ramp_up": false, // If true, GPUs will start mining slowly after DAG generation, then ramp up to full speed.
            
            "cpu_validate": false, // Whether the HTTP API should be enabled or not. Default is enabled
            
            "oc_fan_speed": -1, // set the devices fan speed. -1 = do not change, 0 = auto, >0 = lock at percentage
            
            "oc_power_limit": 0, // set the max power consumption (watts) the device will use. 0 = do not change, >0 = max watts
            
            "oc_core_clock": 0, // set the core clock speed offset in MHz. 0 = do not change, !0 = set clock speed offset
            
            "oc_memory_clock": 0 // set the memory clock speed offset in MHz. 0 = do not change, !0 = set memory speed offset
    
            "cooldown_period": 0, // allow the gpu to cooldown after a period of time. default is 0. Higher value means more time between cooldown periods.   
            
            "force_opencl": 0, // force device to mine on OpenCL (if available, only affects nvidia)
            
            "reset_oc_dag_gen": false // Resets overclocks before DAG is generated. Sets overclocks back after DAG has completely been generated. Useful for high oc when getting a lot of invalid shares due to invalid DAG generation
        },
        {
            "uid": "39:0",
            
            "algorithm": "ethash",
            
            "wallet": "0x0000000000000000000000000000000000000000",

            "pool": [  // default pool address new devices will use. This can be an array or a single string
                "stratum+tcp://yourpool:4444",
                "stratum+tcp://yourpool:4444"
            ],
            
            "pool_username": "rig",
            
            "pool_password": "",
            
            "temp_start": 80,
            
            "temp_stop": 120,
            
            "start_mining": true
            
            "intensity": 8,
            
            "thrash": false,
            
            "stales_ok": false,
            
            "test": false,
            
            "test_diff": 1,
            
            "ramp_up": true,
            
            "cpu_validate": false,
            
            "oc_fan_speed": -1,
            
            "oc_power_limit": 0,
            
            "oc_core_clock": 0,
            
            "oc_memory_clock": 0,
    
            "cooldown_period": 0,
            
            "force_opencl": 0 ,
            
            "reset_oc_dag_gen": false
        }
    ]
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
- `cfg` - The device configuration. eg. i6 = intensity 6
- `free` - Free device memory
- `total` - Total device memory
- `core` - Current core clock speed
- `mem` - Current memory clock speed
- `fan` - Fan speed
- `pwr` - Power Usage
- `temp` - Temperature. In the format of `core temp/memory temp`
- `status` - Device status.

### Supported Pool table columns (In addition to all device table supported columns)
- `uptime` - The time the device has been mining without stopping or resetting
- `a/r/i` - Accepted/Rejected/Invalid shares
- `avg` - Average time between each share
- `eff` - Efficiency, in the units of hashes per watt
- `pool mh` - Estimated effective hashrate based on pool difficulty and shares found (effective/current hashrate on some pools)
- `miner mh` - Current miner speed (reported hashrate on some pools)
- `status` - Device/Pool status.

The pool table has an additional row at the bottom which is the summary of all devices mining to that pool. The status column for the pool includes the epoch number, difficulty, and average latency to pool.

### Customizing the display
- Colors are integers and correspond to normal terminal color codes. Default BzMiner logging color is 7.
- Columns are a comma separated string. do not include spaces unless the column name includes spaces (eg. "pool mh"). Columns can have options (currently just one), which is specified by adding a colon (:) after the column name, and specifying the option, eg `name:nw`. Currently the only option supported is `nw`, which prevents that particular column from wrapping to the next line if the column cannot get wide enough to support the entire value on one line.

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
