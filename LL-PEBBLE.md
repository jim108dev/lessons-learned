# Lessons Learned

This is a learning project in order to summarize the insides gained from several learning projects and internet articles.

- [Lessons Learned](#lessons-learned)
  - [Working environment](#working-environment)
    - [SDK](#sdk)
    - [Add to Fish PATH](#add-to-fish-path)
  - [VS Code Extensions](#vs-code-extensions)
    - [Build and deploy](#build-and-deploy)
  - [C quirks and features](#c-quirks-and-features)
    - [Local variables](#local-variables)

## Working environment

### SDK

Source: [Native SDK install on Linux](https://willow.systems/blog/pebble-sdk-installation-guide/).

1. Installation

   ```sh
   sudo apt update && sudo apt upgrade -y

   sudo add-apt-repository universe
   sudo apt update 
   sudo apt install python2 -y

   curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
   sudo python2 get-pip.py

   sudo apt install wget python-dev libsdl1.2debian libfdt1 libpixman-1-0 npm gcc -y

   mkdir -p ~/opt/pebble-dev

   cd ~/opt/pebble-dev
   wget https://rebble-sdk.s3-us-west-2.amazonaws.com/pebble-sdk-4.6-rc2-linux64.tar.bz2
   tar -jxf pebble-sdk-4.6-rc2-linux64.tar.bz2
   cd ~/opt/pebble-dev/pebble-sdk-4.6-rc2-linux64
   
   bash
   python2 -m pip install virtualenv
   python2 -m virtualenv .env
   source .env/bin/activate
   pip install -r requirements.txt
   deactivate

   echo 'export PATH=~/opt/pebble-dev/pebble-sdk-4.6-rc2-linux64/bin:$PATH' >> ~/.bashrc 
   source ~/.bashrc

   pebble sdk install latest
   ```

### Add to Fish PATH

```sh
set -U fish_user_paths ~/opt/pebble-dev/pebble-sdk-4.6-rc2-linux64/bin $fish_user_paths
echo $PATH
```

## VS Code Extensions

1. "willowsystems.pebble-tool": Run command from the UI. Seams to work only with a single workspace.
1. "ms-vscode.cpptools": C/Cpp support. It was only possible to set the SDK path locally with
   1. `.vscode/c_cpp_properties.json`:

   ```json
      "includePath": [
      "~/.pebble-sdk/SDKs/4.3/sdk-core/pebble/aplite/include/**",
      "${workspaceFolder}/**"
      ],

## Development

### Pair

1. Delete existing pair devices on the watch.

1. Pair from the laptop:

  ```sh
  bluetoothctl
  scan on
  #[NEW] Device B0:B4:48:93:68:71 Pebble 6871
  remove B0:B4:48:93:68:71
  pair B0:B4:48:93:68:71
  #yes
  sudo rfcomm bind 0 B0:B4:48:93:68:71 1
  ```
  
### Build and deploy

```sh
pebble build
# On the emulator
pebble install --emulator=aplite
# On the device
pebble install --serial /dev/rfcomm0
```

## C quirks and features

### Local variables

1. Allays you `malloc` to initialize arrays. Don't do:

  ```c
    success((Record[]){
              {.id = 1,
               .question = "Mock Question",
               .answer = "Mock Answer",
               .choice = 0,
               .last_tested = 50 * 3 * 10 ^ 7},
              ...
          },
          2);
  ```

  The same applies to char arrays. **"Arrays are always passed by reference in C."** See [(Germain)](https://www.cs.utah.edu/~germain/PPS/Topics/C_Language/c_functions.html).
  