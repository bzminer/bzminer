## Why use BzMiner (v4.0)?
- Low dev fee of 0.5%
- Now monitoring memory junction temperature (windows only)
- Awesome Easy to use Linux and Windows GUI miner (Command Line Interface is optional, `bzminer.exe`)
- HTTP API!
- Remote management of all mining rigs on your network
- Fastest Cuda v11 and OpenCL Ethereum/ethash miner
- Control algorithm/pool/wallet mining PER device!
- Constantly being improved based on feedback from you!
- Ability to OC in miner (OC's are removed during DAG generation to prevent invalid DAG generation)
- Advanced mining features, including cooldown periods, thrashing, stales_ok and ramp up 
- DAG Validation for very high OC cards
- AMD Ethash mining (experimental)
- Supported Algos:
    - Ethash (AMD, Nvidia)
    - Etchash (AMD, Nvidia) 


## Current planned major features by version (not including minor releases)
- v5.0 next algo: KawPow (RVN/Ravencoin), autolykos (ERG/Ergo), undecided as of now


## Discord Server
https://discord.gg/NRty3PCVdB


## Requirements
- At least one Nvidia GPU and Nvidia drivers
- Or at least one AMD GPU and AMD drivers

## Quick Start
Launch `bzminer.exe` with the wallet and pool address as parameters:

`bzminer.exe -w 0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388 -p ethstratum+tcp://eth.geodns.flexpool.io:4444`

## Getting Started
There are two ways to run BzMiner. Either through the command line interface (`bzminer.exe`), or through the graphical user interface (`bzminergui.exe`).

There is no need to run both `bzminer.exe` and `bzminergui.exe`, although if `bzminergui.exe` detects `bzminer.exe` is already running, it will not run the miner.

Either launch `bzminergui.exe` and update your wallet and pool address there, or edit `config.txt` and launch `bzminer.exe`.

Optionally you can launch `bzminer.exe` with the wallet, pool url, and rig (worker) name as arguments, such as:

`bzminer.exe -w 0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388 -p ethstratum+tcp://eth.geodns.flexpool.io:4444 -r rigname`


## GUI (bzminergui.exe)
The BzMiner graphical user interface allows you to remotely monitor and manage all instances of BzMiner on your network.

When `bzminergui.exe` is launched, it first checks to see if there's a local instance of BzMiner running (`bzminer.exe` or another `bzminergui.exe`). If it doesn't detect a local instance already running, it will start the miner internally.

![image](https://user-images.githubusercontent.com/83083846/128022812-6b8d4940-ea56-4493-b8b4-dd8ea0112249.png)


## CLI (bzminer.exe)
BzMiner comes with an optional, lighter weight, command line interface, `bzminer.exe`. Simply update `config.txt` and launch `bzminer.exe`.

`bzminer.exe` has an optional parameter, `-c`, which can be called to load a different configuration file. eg. `bzminer.exe -c custom_config.txt`.

`bzminer.exe` also has optional parameters for overriding the pool url, wallet address, algorithm (ethash only supported right now) and rig name. These will be saved in the config for all device overrides

```
>bzminer.exe --help
BZMiner is an enhanced CUDA and OpenCL Ethereum Ethash miner
Usage: bzminer.exe [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -a TEXT                     Default Mining algorithm. eg. 'ethash'
  -r TEXT                     Default Rig (worker/username) name. eg. 'Rig'
  --pool_password TEXT        Default Pool password
  --nvidia INT                Only mine with Nvidia devices (0 = false, 1 = true)
  --amd INT                   Only mine with AMD devices (0 = false, 1 = true)
  -o TEXT                     If provided, output will be logged to this file
  -w TEXT                     Wallet Address
  -p TEXT                     Pool Address
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
  ```

![image](https://user-images.githubusercontent.com/83083846/132102657-c729363e-7a94-46dc-b40f-2b5ba27ac845.png)


## The Configuration File (config.txt)
The configuration file is in the json file format, and can be manually updated, or updated through `bzminer.exe`'s remote commands.

BzMiner reads and saves to the configuration file. Upon first running BzMiner, the configuration file will be updated with a new property called `device_overrides`, which contain a list of all the mining devices found on the pc. This is where individual device settings can be found.

```
{
    "algorithm": "ethash", // default algorithm new devices will use
    
    "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388", // default wallet/username new devices will use
    
    "pool": "stratum+tcp://us1.ethermine.org:4444", // default pool address new devices will use
    
    "pool_password": "", // password to use for pool. Leave blank if no password required
    
    "farm": "farm", // name of the farm this rig belongs to
    
    "rig": "rig", // name of this rig. A.K.A. default worker/username
    
    "nvidia_only": false, // Mine only with Nvidia devices
    
    "amd_only": false, // Mine only with AMD devices
    
    "launch_cli_on_start": false, // whether cli.exe should start on reboot (windows only)
    
    "launch_gui_on_start": false, // whether bzminer.exe should start on reboot. If both are true, only bzminer.exe will start (windows only)
    
    "log_file": "", // file to output logs to. leave blank to disable logging to a file
    
    "verbosity": 2, // log verbosity. 0 = Error, 1 = Warn, 2 = Info (default), 3 = debug, 4 = network
    
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
    
    "device_overrides": [ // list of individual device settings
        {
            "uid": "37:0", // pci bus id : pci device id. this uniquely identifies the device as long as it doesn't change pci slots
            
            "algorithm": "ethash", // algorithm this device should mine using
            
            "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388", // wallet/username this device should use
            
            "pool": "stratum+tcp://us1.ethermine.org:4444", // pool address this device should use
            
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
            
            "force_opencl": 0 // force device to mine on OpenCL (if available, only affects nvidia)
        },
        {
            "uid": "39:0",
            
            "algorithm": "ethash",
            
            "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388",
            
            "pool": "stratum+tcp://us1.ethermine.org:4444",
            
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
            
            "force_opencl": 0 
        }
    ]
}
```


## Pool URLs

Urls should be in the form of `{protocol}+{tcp/ssl}:{url}:{port}` (username and password are optional)

eg. `stratum+tcp://us1.ethermine.org:4444`

If username and password are required, eg. `{protocol}+{tcp/ssl}:<{username}:{password}@>{url}:{port}`, they can be set per device in the config file, or through the command line. If device pool_username is blank, it will use the rig name from the config file. if pool_password is blank, it will use the default pool_password for the rig in the config file. Leave both pool_password blank if no password is required for the pool. Do not put the username and password in the url (If there are enough requests for this, this can be added in a future release)

BzMiner supports 4 network protocols
- Stratum - use `stratum`
- Eth Proxy - use `ethproxy`
- Ethereum Stratum v1.0.0 - use `ethstratum`
- Ethereum Stratum v2.0.0 - use `ethstratum2`

BzMiner will attempt to auto select the protocol if the provided protocol does not succeed in establishing a connection with the pool.


## HTTP API

HTTP API can be enabled by launching bzminer with the --http_enabled flag, or by setting "http_enabled": true in the config file.

For more details, enable the http api, and go to `http://{http_address}:{http port}/help`. If you are on the same computer, you can use `localhost` or `127.0.0.1` as the http_address.

A simple viewer can be found at `/index` when the HTTP API is enabled.
