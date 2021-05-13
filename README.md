## Why use BzMiner (v2.0)?
- Low dev fee of 0.5%
- Awesome Easy to use Linux and Windows GUI miner (Command Line Interface is optional, `bzminer.exe`)
- Remote management of all mining rigs on your network
- Fastest Cuda v11 Ethereum/ethash miner (from limited testing)
- Control algorithm/pool/wallet mining PER device!


## Current planned major features by version (not including minor releases)
- v3.0 OpenCL/AMD Ethereum/Ethash mining (v1.0 Cuda/Nvidia only is supported)
- v4.0 KawPow (RVN/Ravencoin) support


## Discord Server
https://discord.gg/NRty3PCVdB


## Requirements
- At least one Nvidia GPU and Nvidia drivers


## Getting Started
There are two ways to run BzMiner. Either through the command line interface (`bzminer.exe`), or through the graphical user interface (`bzminergui.exe`).

There is no need to run both `bzminer.exe` and `bzminergui.exe`, although if `bzminergui.exe` detects `bzminer.exe` is already running, it will not run the miner.

Either launch `bzminergui.exe` and update your wallet and pool address there, or edit `config.txt` and launch `bzminer.exe`.

Optionally you can launch `bzminer.exe` with the wallet, pool url, and rig (worker) name as arguments, such as:

`bzminer.exe -w 0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388 -p stratum+tcp:us1.ethermine.org -r rigname`


## GUI (bzminergui.exe)
The BzMiner graphical user interface allows you to remotely monitor and manage all instances of BzMiner on your network.

When `bzminergui.exe` is launched, it first checks to see if there's a local instance of BzMiner running (`bzminer.exe` or another `bzminergui.exe`). If it doesn't detect a local instance already running, it will start the miner internally.

![image](https://user-images.githubusercontent.com/83083846/118182675-e893d680-b3fe-11eb-9e85-4b6ace6ac94f.png)


## CLI (bzminer.exe)
BzMiner comes with an optional, lighter weight, command line interface, `bzminer.exe`. Simply update `config.txt` and launch `bzminer.exe`.

`bzminer.exe` has an optional parameter, `-c`, which can be called to load a different configuration file. eg. `bzminer.exe -c custom_config.txt`.

`bzminer.exe` also has optional parameters for overriding the pool url, wallet address, algorithm (ethash only supported right now) and rig name. These will be saved in the config for all device overrides

```
>bzminer.exe --help
BZMiner is an enhanced CUDA Ethereum Ethash miner
Usage: bzminer.exe [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -a TEXT                     Mining algorithm. eg. 'ethash'
  -r TEXT                     Rig (worker) name. eg. 'Rig'
  -o TEXT                     If provided, output will be logged to this file
  -w TEXT                     Wallet Address
  -p TEXT                     Pool Address
  -v INT                      Set log verbosity. 0 = Error, 1 = warn, 2 = info 3 = debug, 4 = network
  -c TEXT                     Config file to load settings from. Default is config.txt
  ```

![image](https://user-images.githubusercontent.com/83083846/116637828-921c9780-a92a-11eb-9ae1-7f87fc651d3a.png)


## The Configuration File (config.txt)
The configuration file is in the json file format, and can be manually updated, or updated through `bzminer.exe`'s remote commands.

BzMiner reads and saves to the configuration file. Upon first running BzMiner, the configuration file will be updated with a new property called `device_overrides`, which contain a list of all the mining devices found on the pc. This is where individual device settings can be found.

```
{
    "algorithm": "ethash", // default algorithm new devices will use
    
    "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388", // default wallet/username new devices will use
    
    "pool": "stratum+tcp://us1.ethermine.org:4444", // default pool address new devices will use
    
    "farm": "farm", // name of the farm this rig belongs to
    
    "rig": "rig", // name of this rig. A.K.A. worker name
    
    "launch_cli_on_start": false, // whether cli.exe should start on reboot (windows only)
    
    "launch_gui_on_start": false, // whether bzminer.exe should start on reboot. If both are true, only bzminer.exe will start (windows only)
    
    "log_file": "", // file to output logs to. leave blank to disable logging to a file
    
    "verbosity": 2, // log verbosity. 0 = Error, 1 = Warn, 2 = Info (default), 3 = debug, 4 = network
    
    "temp_start": 80, // default resume mining once device reaches this temp
    
    "temp_stop": 120, // default pause mining on device if over this temp (120 is basically disable pausing on temp)
    
    "device_overrides": [ // list of individual device settings
        {
            "uid": "37:0", // pci bus id : pci device id. this uniquely identifies the device as long as it doesn't change pci slots
            
            "algorithm": "ethash", // algorithm this device should mine using
            
            "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388", // wallet/username this device should use
            
            "pool": "stratum+tcp://us1.ethermine.org:4444", // pool address this device should use
            
            "temp_start": 80, // resume mining once device reaches this temp
            
            "temp_stop": 120,// dpause mining on device if over this temp
            
            "start_mining": true // whether to start mining on this device when BzMiner starts
            
        },
        {
            "uid": "39:0",
            
            "algorithm": "ethash",
            
            "wallet": "0xBd86b99A0e5eB05cfADB02F82D8a1BFe75d82388",
            
            "pool": "stratum+tcp://us1.ethermine.org:4444",
            
            "temp_start": 80,
            
            "temp_stop": 120,
            
            "start_mining": true
        }
    ]
}
```
