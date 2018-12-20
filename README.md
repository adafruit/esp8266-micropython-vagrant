# DEPRECATED LIBRARY.

This library has been deprecated! We are leaving this up for historical and research purposes but archiving the repository.

We are now only supporting the use of our CircuitPython. Please see this guide on building CircuitPython: https://learn.adafruit.com/building-circuitpython

# ESP8266 & MicroPython SDK Virtual Machine
Vagrant file to build a virtual machine that can compile the [ESP8266 open SDK](https://github.com/pfalcon/esp-open-sdk) &amp; 
[MicroPython](https://micropython.org/) firmware.

Note that MicroPython support for the ESP8266 is in _very_ early stages and does
not support the full capabilities of other MicroPython boards.  However this VM
will help make it easy to build and install MicroPython for the ESP8266 to test
it out and even contribute to it.

Many thanks to the contributors of ESP open SDK & MicroPython for making
their excellent software available!

# Dependencies

You must have the following software installed:

*  [VirtualBox](https://www.virtualbox.org/)
*  [Vagrant](https://www.vagrantup.com/)

# Usage

Clone this repository and navigate to it in a command terminal, then run the
following command to bring the Vagrant virtual machine up and provision it for
compiling the tools:

    vagrant up

This will set up your development environment and will take about 30 minutes to
an hour or more depending on the speed of your machine. This is due to the
process of cloning and compiling three git repositories:

*   [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk) - This is an SDK to
    compile code for the ESP8266's processor.

*   [esp-idf](https://github.com/espressif/esp-idf) - The official Espressif IoT
    development framework.

*   [micropython](https://github.com/micropython/micropython) - This is the MicroPython
    SDK which allows running embedded Python code on an ESP8266.

After the virtual machine is brought up and provisioned, use the following
command to enter an SSH session on it:

    vagrant ssh

Once inside the virtual machine you will see the two git repositories that have
already been cloned.

Note that if the setup fails with an error like:

    [ERROR]    collect2: error: ld terminated with signal 9 [Killed]
    [ERROR]    make[4]: *** [cc1] Error 1
    [ERROR]    make[3]: *** [all-gcc] Error 2

It means that the virtual machine ran out of memory during the last stages of the
compilation process.  You can resolve this by bumping up the memory available to
the VM by changing this line in the Vagrantfile

    # Bump the memory allocated to the VM up to 1 gigabyte as the compilation of
    # the esp-open-sdk tools requires more memory to complete.
    v.memory = 1024

I found at least 1 gigabyte of memory was required to compile the SDK (and that
is the default configuration value).  If you change the Vagrantfile you will
need to stop and restart the VM (see the Stopping & Starting the VM section
further below).

## MicroPython Firmware

After the development environment finishes, a file should have been produced ./build/firmware-combined.bin.
This file should be flashed to the ESP8266 using any convenient flashing tool
(see Flashing ESP8266 Firmware below).  You can copy the firmware-combined.bin file
to Vagrant's shared directory so it is accessible from your main computer and
not just the Vagrant VM.  Do this by running:

    cp ./build/firmware-combined.bin /vagrant/

Now on your machine (not on the VM!) look inside the folder with the
Vagrantfile and you should see the firmware-combined.bin file.

If you make any changes to MicroPython, like modifying the ./scripts/main.py
file to change the boards's behavior on boot, you can recompile the MicroPython
firmware by running these commands again:

    cd ~/micropython/esp8266
    make

## Stopping & Starting the VM

To stop the VM make sure you've exited from any SSH session on it (run the `exit` 
command) and then run this command inside the directory with the Vagrantfile:

    vagrant halt

To start the VM and SSH into it again just run:

    vagrant up
    vagrant ssh

## Flashing ESP8266 Firmware

To flash the MicroPython firmware to the ESP8266 you can use the excellent
[esptool.py](https://github.com/themadinventor/esptool) Python script.  Note that
you'll run this script from your machine and _not_ the Vagrant VM that compiled
the firmware!

First you need to make sure you have [Git](http://git-scm.com/downloads) 
and [Python 2.7](https://www.python.org/downloads/) installed and have the 
[PySerial library](http://pyserial.sourceforge.net/).  For Windows users 
the easiest thing to do is install the [pyserial-2.7.win32.exe installer](https://pypi.python.org/pypi/pyserial) 
and run it to install the library.  For Mac OSX or Linux users you can 
instead install the library by [installing pip](https://pip.pypa.io/en/latest/installing.html) 
and then running in a terminal:

    sudo pip install pyserial

Once pyserial is installed clone the esptool.py repository by running in
a terminal:

    git clone https://github.com/themadinventor/esptool.git

Then change to the directory with the esptool.py script and invoke it with the -h
option to see its usage:

    cd esptool
    python esptool.py -h

You should see something like the following printed:

    usage: esptool [-h] [--port PORT] [--baud BAUD]
               {load_ram,dump_mem,read_mem,write_mem,write_flash,run,image_info,make_image,elf2image,read_mac,flash_id,read_flash,erase_flash}
                   ...
    ...

Now to flash the ESP8266 firmware make sure your ESP8266 is connected to your
machine using a USB to serial cable.  Find the name of the serial port using
Device Manager on Windows, or running `ls -l /dev/tty*` on Mac OSX or Linux
(usually it's a device /dev/ttyUSB on Linux).

You also need to make sure you have a firmware-combined.bin file that was
built inside the VM in the previous steps.

To flash the chip with the firmware, hold down the GPIO0 button and then press
the reset button (while still holding GPIO0).  Release the reset button and then
release the GPIO0 button.  Now run in the terminal:

    python esptool.py -p <serial port name> write_flash 0x00 firmware-combined.bin

Replace `<serial port name>` with the name of the serial port connected to the
ESP8266, and replace firmware-combined.bin with the path to the firmware-combined.bin
file if it is not already in the same directory.  For example to flash the chip
from a Linux machine a command like the following is run:

    python esptool.py -p /dev/ttyUSB0 write_flash 0x00 firmware-combined.bin

The ESP8266 will be flashed with the MicroPython firmware and you should see output
like the following:

    Connecting...
    Erasing flash...
    Writing at 0x0004d800... (100 %)
    
    Leaving...

Congratulations you've flashed the ESP8266 with MicroPython!  Now to test it out
connect to the ESP8266's serial port at 115200 baud.  You should see a Python REPL,
for example try typing:

    print('Hello world!')
