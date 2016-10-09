# AM Monitor (GENIVI Audio Manager Monitor)

> Authors: JoonCheol Park <jooncheol.park@windriver.com>, Daewon Park <daewon.park@windriver.com>

---
## Introduction

> The AM Monitor shows to working flow of the AM. In this demo, we use the AM with 2 Plugins for PulseAudio and Audio Policy Control. When we play the audio in this app, we can see the internal status of connections in AM and status of the PulseAudio.

---
## Dependencies

* Audio Manager (branch Intreprid_stable_branch, Version 6.1)
 + 2 patches for PulseAudio Routing Plugin and Audio Policy Control Plugin
    (http://git.projects.genivi.org/?p=meta-genivi-demo.git;a=tree;f=recipes-multimedia/audiomanager/audiomanager;h=f134ecaf6190ec272bd6136ee88f0e86a8b86047;hb=HEAD)
* PulseAudio
* DBus
* Qt5
* Python
* sqlite3

---
## Build Instruction for Ubuntu 14.04 PC


### Build Audio Manager + 2 patches

    $ git clone git://git.projects.genivi.org/AudioManager.git
    $ curl "http://git.projects.genivi.org/?p=meta-genivi-demo.git;a=blob_plain;f=recipes-multimedia/audiomanager/audiomanager/0001-Porting-Pulse-Routing-Interface-from-AM-v1.x-to-AM-v.patch;h=7d6ebbf710d0a0d46d99797f5ef6dd3712a888fe;hb=HEAD" > 0001-Porting-Pulse-Routing-Interface-from-AM-v1.x-to-AM-v.patch
    $ curl "http://git.projects.genivi.org/?p=meta-genivi-demo.git;a=blob_plain;f=recipes-multimedia/audiomanager/audiomanager/0001-Porting-Pulse-Control-Interface-from-AM-v1.x-to-AM-v.patch;h=9a61d3c83cc5954bca1a0c644b3aec4aa94e8f5d;hb=HEAD" > 0001-Porting-Pulse-Control-Interface-from-AM-v1.x-to-AM-v.patch
    $ cd AudioManager
    $ git checkout 6e700bfc7faecb6d7e0f31c9d08b8b4f6cd1b3dd
    $ patch -p1 < ../0001-Porting-Pulse-Routing-Interface-from-AM-v1.x-to-AM-v.patch
    $ patch -p1 < ../0001-Porting-Pulse-Control-Interface-from-AM-v1.x-to-AM-v.patch
    $ mkdir build
    $ cd build
    $ cmake -DWITH_PULSE_ROUTING_PLUGIN=ON -DWITH_PULSE_CONTROL_PLUGIN=ON -DWITH_ENABLED_IPC=DBUS -DWITH_DATABASE_STORAGE=OFF -DWITH_DLT=OFF -DCMAKE_INSTALL_PREFIX=(PREFIX on your dev environment) ..
    $ make
    $ make install


### Build AM Monitor

    $ git clone git://git.projects.genivi.org/AudioManagerDemo.git
    $ qmake
       or
    $ qmake INCLUDEPATH=(PREFIX-OF-AM)/include
    $ make

 If you have a trouble with the compile due to not enough memory (e.g. build on VMWare), You can comment out the line "RESOURCES += qml.qrc" from AudioManager.pro. The executable binary won't have big resouce files in it.

    $ vi AudioManagerMonitor.pro # comment out line "RESOURCES += qml.qrc"
    $ qmake && make
    $ ./AudioManagerMonitor --debug # run with --debug to load resources directly from filesystem.

 Or you can add swap file temporarily for enough memory space for build on VM.

    $ dd if=/dev/zero of=/swapfile bs=1K count=2000000 # 2G, you can try more
    $ mkswap /swapfile
    $ swapon /swapfile
    $ # if you want to use this for next boot, add "/swapfile none swap sw 0 0" in /etc/fstab and reboot

---
## Running AM Monitor on Ubuntu 14.04 PC

1. Set testing environment

    $ . pc-env

  This script creates 2 virtual sink device for PulseAudio Daemon. You can check with "pacmd list-sinks". If you want to remove created sink device, execute 'deactivate' on the shell

2. Execute AM

    $ AudioManager -V -c(PREFIX-OF-AM)/lib/audioManager/control/libPluginControlInterface.so -l(PREFIX-OF-AM)/lib/audioManager/command/ -r(PREFIX-OF-AM)/lib/audioManager/routing/

3. Execute AM Monitor

    $ ./AudioManagerMonitor --debug

 If you don't use any argument, AM Monitor will be running as fullscreen mode. Please refer the usage with "--help" option.

---
## Audio Policy Rule Configuration example

* Preset of AM Sources and Sinks
  (PREFIX-OF-AM)/lib/audiomanager/routing/libPluginRoutingInterfacePULSE.conf
  It defines 5 preset of AM Sources - Entertainment, Navigation, TTS ...
* Policy Rule
  (PREFIX-OF-AM)/lib/audiomanager/control/libPluginControlInterface.conf
  It defines audio policy scenario. E.g - When play the NAVI sound alert during Media playing, Sink Volume of previous Media goes down to 50%.

---
## To do items

* AM Monitor
 * Use CommonAPI for command interface in AM Monitor App

* AM Plugins
 * Enhance the Routing Plugin for PulseAudio
   * Implement missing code for all Routing Plugin APIs
   * Enhance the configuration syntax
 * Enhance the Conrol Plugin for Audio Policy
   * More flexible and programmable audio policy configuration using JavaScript
