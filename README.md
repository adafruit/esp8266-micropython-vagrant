# ESP8266 & MicroPython SDK Virtual Machine
Vagrant file to build a virtual machine that can compile the ESP8266 open SDK &amp; 
MicroPython firmware.

# Dependencies

You must have the following software installed:

*  [VirtualBox](https://www.virtualbox.org/)

*  [Vagrant](https://www.vagrantup.com/)

# Usage

Clone this repository and navigate to it in a command terminal, then run the
following command to bring the Vagrant virtual machine up and provision it for
compiling the tools:

    vagrant up

After the virtual machine is brought up and provisioned use the following
command to enter an SSH session on it:

    vagrant ssh

Once inside the virtual machine you will see two git repositories that have
already been cloned:

*   [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk) - This is an SDK to
    compile code for the ESP8266's processor.

*   [micropython](https://github.com/micropython/micropython) - This the MicroPython
    SDK which allows running embedded Python code on an ESP8266.

## ESP Open SDK Compilation

You will want to first compile the ESP open SDK by executing (note that the
compilation will take about 30 minutes to an hour or more depending on the speed
of your machine):

    cd esp-open-sdk
    make STANDALONE=y

Note that if the compilation fails with an error like:

    [ERROR]    collect2: error: ld terminated with signal 9 [Killed]
    [ERROR]    make[4]: *** [cc1] Error 1
    [ERROR]    make[3]: *** [all-gcc] Error 2

This means the virtual machine ran out of memory during the last stages of the
compilation process.  You can resolve this by bumping up the memory available to
the VM by changing this line in the Vagrantfile

    # Bump the memory allocated to the VM up to 1 gigabyte as the compilation of
    # the esp-open-sdk tools requires more memory to complete.
    v.memory = 1024

I found at least 1 gigabyte of memory was required to compile the SDK (and that
is the default configuration value).  If you change the Vagrant file you will
need to stop and restart the VM (see the Stopping & Starting the VM section
further below).

Once the ESP open SDK compilation has finished you need to extend the path
environment variable so the tools are accessible to MicroPython and other build
scripts.  You can do this by changing the .profile file so that on every login
to the virtual machine it extends the path to includes these tools.  Do this by
executing:

    echo "PATH=$(pwd)/xtensa-lx106-elf/bin:\$PATH" >> ~/.profile

Finally exit and re-enter the VM to make this path change available by running:

    exit
    vagrant ssh

## MicroPython Compilation

After the ESP open SDK has been compiled and added to the path, execute the
following commands to start compiling MicroPython (the compilation is quick and
only takes a few minutes depending on the speed of your machine):

    cd ~/micropython/unix
    make

Now run the tests included in micropython to help confirm it has been successfully
compiled.  You should see over 400 tests are run and all succeed (if there are
failures search on the micropython github issues to see if they are expected).

    make test

Next install MicroPython's shared tools by running:

    sudo make install

Finally you're ready to compile MicroPython firmware for the ESP8266 by executing:

    cd ~/micropython/esp8266
    make

After the firmware compilation finishes the output will be the file ./build/firmware-combined.elf.
This file should be flashed to the ESP8266 using any convenient flashing tool
(instructions are further below).  You can copy the firmware-combined.elf file
to Vagrant's shared directory so it is accessible from your main computer and
not just the Vagrant VM.  Do this by running:

    cp ./build/firmware-combined.elf /vagrant/

Now on your machine machine (not on the VM!) look inside the folder with the
Vagrantfile and you should see the firmware-combined.elf file.

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
