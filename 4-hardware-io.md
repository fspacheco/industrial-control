# Hardware Connections (Physical I/O)

## Agenda

1.  Beckhoff I/O System Overview
    - Terminal concept (modular I/O)
    - Wiring, configuration, testing
2.  Digital Inputs: EL1008 + Switches
3.  Digital Outputs: EL2008 + LEDs
4.  Analog Inputs: EL3074 + LM35
    - Sensor basics, scaling, engineering units
5.  Mapping GVL to Physical I/Os
    - Best practices for maintainable code
6.  Testing

## Beckhoff I/O Architecture

```
[Industrial PC/Controller] --EtherCAT-->  [Terminals] -
                                                      |
                                                      +-- EL1008 (Digital In)
                                                      +-- EL2008 (Digital Out)
                                                      +-- EL3074 (Analog In)
                                                      +-- EL4074 (Analog Out)
                                                    
```

**Key Concepts:**
- Fieldbus: is an industrial network system for real-time distributed control. There are many options for automation:
    - MODBUS, 1979, by Modicon (now Schneider Electric)
    - PROFIBUS, 1987, German government, Siemens
    - CAN, 1980s, Bosch
- EtherCAT (Ethernet for Control Automation Technology) is an Ethernet-based fieldbus system developed by Beckhoff Automation. The protocol is standardized in IEC 61158.
- Terminals: Snap-on modules, configurable in software. Check the list at https://www.beckhoff.com/en-en/products/i-o/ethercat-terminals/

## Our setup at HAMK ICT Robotics

![](img/hardware/hamk-setup.svg)

### Connections to do

We will connect:

- 2 digital inputs to EL1008 (blue wires in the terminal block 1) from switches 1 and 4
- 3 digital outputs from EL2008 (white wires in the terminal block 1) to the 3 LEDs in the right (yellow, blue, green)

![](img/hardware/terminal-blocks-connection.svg)

Also:
- temperature sensor LM35
    - brown to +24V
    - black to 0V
    - blue to terminal block

The other side of the terminal block connected to the signal (blue) goes to the analog input 1 (banana connector).

![](img/hardware/connection-lm35-sensor.svg)

> :warning: :exclamation: Once finished, ask the teacher to check the connections. Wait for the approval before powering the module.

## Power up

After the approval, you simply connect the power plug. The IPCs have no switch button.

## Network setup

As we did with the Dobot MG400, the computer connected to the Beckhoff IPC should have a fixed IP 192.168.1.4.

![](img/hardware/network-setup.png)

The IP addresses of the Beckhoff IPCs were configured to 192.168.1.10 and 192.168.1.11 (there are two network connections).

Open a shell in your computer and confirm you have network connection to the IPC using `ping`.

![](img/hardware/ping-ipc.png)

## TwinCAT XAE

Our IPCs are running with the TwinCAT Runtime 4024. For this reason, you should open the TwinCAT XAE (yellow icon), not the 64-bit version (that works only with the runtime 4026).

![](img/hardware/windows-tray-twincat-xae.png)

When you open the TwinCAT XAE, the first thing to change is the build version to 4024.35.

![](img/hardware/change-build-4024.png)

<!--![](img/hardware/confirm-4024.png)-->

Then, create a new project in the menu File/New/Project as we did in previous classes.

In the solution, you can keep only the items SYSTEM, PLC, IO, and hide the other ones.

### Connecting TwinCAT XAE to the IPC

Your current target is `<Local>`. We will search for the IPC in the network and establish a connection. Click on `<Local>`. A new window named "Choose Target System" will open. Click on "Search (Ethernet)".

![](img/hardware/from-local-to-search-ipc.svg)

Then, in the "Add Route Dialog", enter the IP address 192.168.1.10 and press enter.

![](img/hardware/enter-ip-to-search.png)

It will find the IPC. If the dialog is not as shown below, choose "Advanced settings". Then change "Address info" to "IP Address" and click on "Add Route".

![](img/hardware/add-route-dialog.svg)

Keep the Secure ADS and Self Signed Certificate options. Add the password. The default is simply the number 1.

![](img/hardware/administrator-password.png)

The target system should show the IPC host name (in this example, BNT-000t4zre) and the lock icon, meaning connection is established.

![](img/hardware/ipc-as-target.png)

### Scanning devices

As discussed before, the IPC is connected to I/O modules. They do not appear automatically in TwinCAT XAE. In IO/Devices, choose "Scan"

![](img/hardware/scan-devices.png)

You can select only the Device 1, since we do not have any module connected to the Ethernet adapters.

![](img/hardware/choose-device-1-ethercat.png)

Confirm you want to scan boxes.

![](img/hardware/confirm-scan-boxes.png)

After some seconds, the panel Solution Explorer in the left will be populated with all the devices found.

![](img/hardware/devices-populated-after-scan.png)

If you click on Device 1, in the General tab you will see the topology of the modules. The column E-Bus (mA) shows the available current. You can see it reduces after each module. When it is too low, you need to add an intermediate potential supply terminal (like [EL9100](https://www.beckhoff.com/en-en/products/i-o/ethercat-terminals/el-ed9xxx-system/el9100.html)) if you want to connect more modules.

![](img/hardware/device-topology-current-ebus.png)

The information about current consumption for each module is available in the manual. For the [EL1008](https://www.beckhoff.com/en-en/products/i-o/ethercat-terminals/el-ed1xxx-digital-input/el1008.html), it is 90 mA.

![](img/hardware/el1008-technical-data.png)

You can confirm visually that you have exactly the same topology shown in the General tab of Device 1.

![](img/hardware/el-terminals.jpg)

## Testing inputs and outputs online

Now, let's test the inputs and outputs without any program running yet. Let's start with the digital outputs in terminal EL2008. Click with right button on EL2008/Channel 1/Output. On the menu, select "Online Write '1'".

![](img/hardware/el2008-channel1-write.png)

The LED on the terminal should turn on, as well as the LED in the pushbutton.

![](img/hardware/el-terminals-digital-output-1-on.svg)

For the channel 2, let's try another way to write values. Double click on EL2008/Channel 2/Output. In the Online tab in the right, note the graph with the value zero. Then, click on "Write".

![](img/hardware/el2008-channel2-online.png)

In the "Set Value Dialog", set the value to 1 and press OK.

![](img/hardware/el2008-channel2-write.png)

You will note that the graph changes to 1, the LED on the EL2008 terminal turns on and also the LED on the pushbutton.

![](img/hardware/el2008-channel2-online-after-write.png)

Now, let's check the inputs that are connected to the switches. Double click on EL1008/Channel 1/Input.

![](img/hardware/el1008-channel1-online.png)

Then, physically change the position of the first switch (the one in the left). You will see the corresponding change in the Online graph of the input.

![](img/hardware/el1008-channel1-online-after-changing-switch.png)

Do the same for the Channel 2, that is, open the Online view. Channel 2 is connected to the last switch (a momentary one), the rightmost one. Change its position and check the graph.

![](img/hardware/el1008-channel2-online-after-switching.png)

Last but not least, let's check the analog inputs in terminal EL3074. Click on the Term 4 (El3074) to open the General tab. You will see values are INT (integers) or as REAL32 (float values). This terminal is very configurable. We will set only two parameters:
1. Values as REAL32 (if it is not showing already)
2. Input range from 0 to 10 V

> :warning: Be careful to not change other parameters.

> :exclamation: The maximum input voltage for this terminal is 10.7 V.

![](img/hardware/el3074-online-int-value.png)

In the tab Process Data, select "Standard (Real32)".

![](img/hardware/EL3074-PDO-change-to-real32.png)

In the tab CoE - Online, search for the index 800D:11 Input Interface. It is probably set as voltage mode, from -10 to +10 V (V ±10V). Click on it. A dialog will open.

![](img/hardware/800D-input-interface-plusminus-10V.png)

In the Set Value Dialog, select V 0-10V.

![](img/hardware/800D-input-interface-change-to-0-10V.png)

You connected th Channel 1 of the EL3074 to a temperature sensor with 3 wires (+24V, signal, 0V).This temperature sensor is the [LM35](https://www.ti.com/lit/ds/symlink/lm35.pdf). It is calibrated to output voltage in a linear scale of 10 mV/°C. It means that if the temperature is around 22°C, you should read 0.22 V (220 mV). Check the value and make it change (if you hold the sensor with your hands, the temperature will increase).

### Program in ST with physical inputs and outputs

To have physical inputs and outputs in your program, first you need to define variables as physically attached to I/O. You do it in ST with the qualifiers:
- `AT %I*` for inputs
- `AT %Q*` for outputs

We will start with the basic program of the refrigerator. When the door is open, the light turns on. Create a global variable list named GVL_IO and add the variables `bDoorOpen` and `bLight`.

![](img/hardware/gvl-io-fridge1.png)

The code is one-line only. Create a POU *fridge1* and add the line `GVL_IO.bLight := GVL_IO.bDoorOpen;`

![](img/hardware/program-fridge1.png)

Then, compile the program (Build Solution).

![](img/hardware/build-solution.png)

Now, we are going to map the physical I/Os to the variables. Right click ELK1008/Channel 1/Input and open "Change Link".

![](img/hardware/el1008-channel1-change-link.png)

A dialog "Attach Variable Input" will open and a list of input variables that can be connected to the physical input will be shown (in this case, it is only one, `bDoorOpen`). Select it.

![](img/hardware/el1008-channel1-change-link-attach-variable.png)

Now, do a similar procedure for the output. Map the El2008/Channel **2**/Output to `bLight`.

![](img/hardware/el2008-channel2-change-link.png)

Note that an I/O mapped to a variable has a small mark (gray arrow on a white background). Only the Channel 2 in EL2008 is mapped.

![](img/hardware/not-linked-vs-linked-channels.png)

The mapping also appears on the instance. You can confirm your two variables are linked to I/Os if they have the gray arrow on a white background. 

There is also another way to link variables and I/Os that we will use in the future, using `attribute 'TcLinkTo'`. Then, the [mapping](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_plc_intro/3107974923.html&id=) will appear as the gray arrow on a blue background.

![](img/hardware/linked-variables-from-instance-view.png)

You can finally transfer the program to the IPC with "Activate Configuration".

![](img/hardware/activate-configuration.png)

 Then, change to runtime mode, log in the IPC and change the switch 1. The LED in the pushbutton should turn on. You can monitor the status of the variables also in the Online mode of the GVL_IO.

![](img/hardware/online-debug-view.png)

### Activity

Implement, step by step, the other versions of the refrigerator project.