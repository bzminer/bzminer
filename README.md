## Why use BzMiner (v1.0)?
- Low dev fee of 0.5%
- Awesome Easy to use Windows GUI miner (Command Line Interface is optional, `cli.exe`)
- Remote management of all mining rigs on your network
- Fastest Cuda v11.2 Ethereum/ethash miner (from limited testing)
- Control algorithm/pool/wallet mining PER device!


## Current planned major features by version (not including minor releases)
- v2.0 OpenCL/AMD Ethereum/Ethash mining (v1.0 Cuda/Nvidia only is supported)
- v3.0 Linux support
- v4.0 KawPow (RVN/Ravencoin) support


## Discord Server
https://discord.gg/sh8aNUWAYA


## Requirements
- Cuda v11.2 Runtime


## Getting Started
There are two ways to run BzMiner. Either through the command line interface (`cli.exe`), or through the graphical user interface (`bzminer.exe`).

There is no need to run both `cli.exe` and `bzminer.exe`, although if `bzminer.exe` detects `cli.exe` is already running, it will not run the miner.

Either launch `bzminer.exe` and update your wallet and pool address there, or edit `config.txt` and launch `cli.exe`.


## GUI (bzminer.exe)
The BzMiner graphical user interface allows you to remotely monitor and manage all instances of BzMiner on your network.

When `bzminer.exe` is launched, it first checks to see if there's a local instance of BzMiner running (`cli.exe` or another `bzminer.exe`). If it doesn't detect a local instance already running, it will start the miner internally.

![image](https://user-images.githubusercontent.com/83083846/116612543-4b647880-a8fd-11eb-89d4-7394c5ca71f6.png)


## CLI (cli.exe)
BzMiner comes with an optional, lighter weight, command line interface, `cli.exe`. Simply update `config.txt` and launch `cli.exe`.

`cli.exe` has an optional parameter, `-c`, which can be called to load a different configuration file. eg. `cli.exe -c custom_config.txt`.

![image](https://user-images.githubusercontent.com/83083846/116620914-17db1b80-a908-11eb-9e8c-6beb6c2b9ffa.png)


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
    
    "launch_cli_on_start": false, // whether cli.exe should start on reboot
    
    "launch_gui_on_start": false, // whether bzminer.exe should start on reboot. If both are true, only bzminer.exe will start
    
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
