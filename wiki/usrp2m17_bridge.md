# USRP2M17 Bridge

To set up an analog to digital M17 bridge, you will need the following:
  * An AllstarLink node (henceforth called ASL in this document)
  * Shell access to the ASL node (SSH, or local terminal)
  * Superuser access on the ASL node
  * git and basic make tools installed on the ASL node
  * An M17 Reflector to connect to

  Assumptions made in this tutorial:
  * You know what an ASL node is, and you have an ASL node available to make changes on
  * You know what the chan_usrp module in ASL is, and know how to configure it
  * You have a working knowledge of Linux and shell commands
  * You have a working knowledge of git, and how to build source code to create an executable binary
  * You're not afraid of breaking things, and you know where to find help online should something break

---

## CAVEAT EMPTOR

This is beta software. Use at your own risk. No warranty is expressed, implied, or given should you break something. If something does break, you get to keep the pieces. If your system catches on fire, call the fire department. No support is given or should be expected from the development team, or the writers of this document. Compiling source code and changing configuration files comes with risk. Following this document, you acknowledge, accept, and take responsibility for those risks. Changing an existing working system to implement new features should not be undertaken by the faint-hearted. Wash in warm water, rinse cold. Hang to dry.

**Do not link analog repeater systems to M17 Reflectors without expressed permission from both sides of the bridge you create.**

### YOU HAVE BEEN WARNED

---

## Network Diagram

```
       127.0.0.1:32001                  0.0.0.0:32010  
     <------------------          <-----------------------
ASL       chan_usrp      USRP2M17       M17 Protocol        M17 Reflector
     ------------------>          ----------------------->
       127.0.0.1:34001              [REFLECTOR IP]:17000
```

---

## Start: USRP2M17

### Download and Compile USRP2M17

First, let's log into the ASL node and create a folder to contain the source code, and change to that directory.

```bash
mkdir git
cd git
```

Next, clone the nostar MMDVM_CM fork to your local computer. Even though we will only be working on one section of the MMDVM_CM code, we need to clone the whole shebang.

```bash
git clone https://github.com/nostar/MMDVM_CM.git
```

Change directories to the USRP2M17 code which we will be compiling.

```bash
cd MMDVM_CM/USRP2M17
```

Now, use `make` to compile the code into a useful binary.

```bash
make
```

If there are no errors, this step is finished and you may proceed to the next. If you do encounter errors - `STOP` - and resolve them before continuing.

### Create a logical directory structure

After the USRP2M17 code is successfully compiled, we now create a places to put the program, its configuration file, and log files.

```bash
sudo mkdir /opt/USRP2M17
sudo mkdir /var/log/usrp
```

Now, move the program and its configuration file to the correct folder.

```bash
sudo cp USRP2M17 /opt/USRP2M17
sudo cp USRP2M17.ini /opt/USRP2M17
```

### Create a daemon user to run USRP2M17

Let's create a daemon user to run the software.

```bash
sudo useradd -M usrp
sudo usermod -L usrp
```

Now, we need to change permissions on the folders we created.

```bash
sudo chown -R usrp:usrp /opt/USRP2M17
sudo chown -R usrp:usrp /var/log/usrp
```

### Edit the USRP2M17 configuration file

You will need to change the configuration file that ships with USRP2M17 to fit your parameters.

```bash
sudo nano /opt/USRP2M17/USRP2M17.ini
```

You will need the following information:
  * Your call sign
  * The name/designator of the M17 reflector, and the module you wish to connect to
  * The IP address of the M17 reflector
  * The directory you will place your log files in

Example: (Do not copy/paste this configuration, edit your file)

```
[M17 Network]
Callsign=AD8DP D                # Change this to your callsign, optionally add a suffix
Address=192.168.1.4             # Change this to the IP address of the reflector you are connecting to
Name=M17-BRO C                  # Change this to the reflector name, designator, and module letter you are connecting to
LocalPort=32010
DstPort=17000
GainAdjustdB=3
Daemon=0
Debug=1

[USRP Network]
Address=127.0.0.1
DstPort=32001
LocalPort=34001
GainAdjustdB=-6
Debug=1

[Log]
# Logging levels, 0=No logging  
DisplayLevel=1
FileLevel=1
FilePath=.                      # Change this to your logging folder
FileRoot=USRP2M17
```

Now that the configuration is created, test it by running the USRP2M17 program.

```bash
sudo -u usrp '/opt/USRP2M17/USRP2M17 /opt/USRP2M17/USRP2M17.ini'
```

If there are no errors, this step is finished and you may proceed to the next. If you do encounter errors - `STOP` - and resolve them before continuing.

### Change USRP2M17 to run as a daemon, and create a systemd unit file

Now that USRP2M17 is running as expected, you can change the configuration file to run the program as a daemon. Find the line in the configuration file that reads:

```
Daemon=0
```

Change the 0 to a 1 on that line in the configuration.

Now, create a new systemd unit file for USRP2M17

Example: (you may copy this example to use on your own system)

```
[Unit]
Description=USRP2M17 Service

[Service]
Type=simple
User=usrp
Restart=on-failure
RestartSec=3
ExecStart=/opt/USRP2M17/USRP2M17 /opt/USRP2M17/USRP2M17.ini
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=multi-user.target
```

After the unit file is created, reload systemd

```
sudo systemctl daemon-reload
```

Then, start the USRP2M17 service

```
sudo systemctl start usrp2m17.service
```

Check to see that the service is running

```
sudo systemctl status usrp2m17.service
```

If there are no errors, this step is finished and you may proceed to the next. If you do encounter errors - `STOP` - and resolve them before continuing.

```bash
sudo systemctl enable usrp2m17.service
```

## End: USRP2M17

---

## Start: ASL

### Some relevant information

It is now time to edit our ASL node. There are three things that you will need to do:
  * Edit rpt.conf to include the node number you will be using for the chan_usrp module
  * Edit modules.conf to load the chan_usrp module
  * Edit extensions.conf to make ASL aware of the new node, and route the call to the correct path

The information for these steps are adapted from many documents available on the Internet, most relevant information comes from the DVSwitch group. Thank you to Steve N4IRS for providing this information.

### Edit rpt.conf

Open the rpt.conf file for editing

```bash
sudo nano /etc/asterisk/rpt.conf
```

Above your existing node configuration, add in the configuration for a new node. Best practice usually is picking a node number between 1900-1999. The following example is for 1999.

Example: (you may copy and paste this into your configuration)

```
[1999]
rxchannel = USRP/127.0.0.1:34001:32001  ; Use the USRP channel driver. Must be enabled in modules.conf
                                        ; 127.0.0.1 = IP of the target application
                                        ; 34001 = UDP port the target application is listening on
                                        ; 32001 = UDP port ASL is listening on

duplex = 0                              ; 0 = Half duplex with no telemetry tones or hang time. Ah, but Allison STILL talks!

hangtime = 0                            ; squelch tail hang time 0
althangtime = 0                         ; longer squelch tail hang time 0

holdofftelem = 1                        ; Hold off all telemetry when signal is present on receiver or from connected nodes
                                        ; except when an ID needs to be done and there is a signal coming from a connected node.

telemdefault = 0                        ; 0 = telemetry output off. Don't send Allison to DMR !!!!!!!!!!!!!!!!! Trust me.

telemdynamic = 0                        ; 0 = disallow users to change the local telemetry setting with a COP command,

linktolink = no                         ; disables forcing physical half-duplex operation of main repeater while
                                        ; still keeping half-duplex semantics (optional)

nounkeyct = 1                           ; Set to a 1 to eliminate courtesy tones and associated delays.

totime = 180000                         ; transmit time-out time (in ms) (optional, default 3 minutes 180000 ms)

; idrecording = |ie                     ; id recording or morse string see http://ohnosec.org/drupal/node/87
; idtalkover = |ie                      ; Talkover ID (optional) default is none see http://ohnosec.org/drupal/node/129
```

You also need to add in the new node number into the nodes stanza in the configuration

```
1999 = radio@127.0.0.1:4569/1999,NONE   ; ADD this to your existing [nodes] stanza
                                        ; 4569 is the default iax port number
```

Save and close the file.

### Edit modules.conf

You now need to enable the loading of the chan_usrp module in Asterisk. Open the modules.conf file

```bash
sudo nano /etc/asterisk/modules.conf
```

Find the line for the chan_usrp module and change it so it reads like so

```
load => chan_usrp.so ; GNU Radio interface USRP Channel Driver
```

You should only need to delete the letters `no` at the beginning of the line. Save and close the file.

### Edit extensions.conf

You now need to let asterisk know how to route the call to your new chan_usrp node. Open the extensions.conf file for editing

```bash
sudo nano /etc/asterisk/extensions.conf
```

Find the globals stanza, and make a new variable for your node number. The example provided shows what the whole block should look like

```
[globals]
HOMENPA = 999                        ; change this to your Area Code
NODE = [YOUR EXISTING NODE NUMBER]   ; change this to your node number
NODE1 = 1999                         ; add in your new node here
```

In the radio-secure stanza, add in a line for your new node

```
[radio-secure]
exten => ${NODE},1,rpt,${NODE}       ; your existing node
exten => ${NODE1},1,rpt,${NODE1}     ; your new chan_usrp node
```

Save and close the file.

### Restart Asterisk

Now that your configuration is complete, you must restart Asterisk. Different ASL distros have different ways of doing this, but for example, you could try

```bash
sudo astres.sh
```

### Connect your new node to your existing node

You can do this one of two ways, DTMF from a radio to your node, or through the Asterisk CLI.

For DTMF, key your radio and dial `*3` followed by your chan_usrp node number. For the 1999 example, dial `*31999`

You should then hear Allison say `Node 1 9 9 9 connected to [YOUR NODE NUMBER]`

If you wish to do this from the Asterisk CLI, use the following command (using 1999 as the example again)

```
rpt fun [YOUR NODE NUMBER] *31999
```

You should then hear Allison say `Node 1 9 9 9 connected to [YOUR NODE NUMBER]`

Congratulations! You should now be able to key up an analog radio or dial into your node, and then be heard on the M17 Reflector module that you chose in the configuration. Pat yourself on the back, you smart person, you!

### Optionally

Set a startup macro in rpt.conf to automatically connect to the node running chan_usrp. Find the line in rpt.conf that starts with `startup_macro = ` and add in the command to dial the node. Example given for node 1999

```
startup_macro = *31999
```

## End: ASL