## Notes for M1 Mac and other non x64 machines

---

The M1 Mac and other non x64 systems will not be able to directly run the provided virtual machine images
(at least at the time of writing).
VMWare is working on support emulating x64 on ARM, but in the meanwhile you need to perform some adjustments yourself.

#### Tip for Networking

This is a bit different in UTM. The default network driver is Shared Network so the VM will get its own IP.
This will change the Week 5 Jupyter address from  `localhost:8888` to something like `http://192.168.64.4:8888/`.
You can find out the ip address with `ip address`.

## Creating M1 Compatible Images

---

This will only work with images coming in `.ova` format so the Kali VM will not work `.vmwarevm`.
The images can only consist of one `.vmdk` image, otherwise the scripts need some changes.
This uses around 3-4 times the image size of disk space while converting so make sure you have enough.

### Using provided Docker Image

Fairly straightforward and should work on any linux based machine with Docker

#### Required Software

- Docker [install link](https://docs.docker.com/desktop/install/mac-install/)
- `tar` command (should be available by default)

#### Converting the VM images

Do this for both course VM images

1. Download both the Week 1-4,6 and Week 5 VMs in __OVA__ format
1. __Move__ the VM `.ova` image to this folder
2. Run `sh create-qcow.sh image_name.ova`
    1. Replace `image_name` with the actual name
3. You will get the `image_name.ova.qcow2` as output

### Alternative Approach

This might run a bit faster, but will be more annoying to set up if you do not have the pre-requisites

#### Required software

- UTM Virtualization/Emulation Software [install link](https://mac.getutm.app/)
    - Downloading from the page is free
    - Install this before starting anything else
- `tar` command
- `qemu-img` command (via homebrew etc.)

#### Converting the VM images

Do this for both course VM images

1. Download both the Week 1-4,6 and Week 5 VMs in __OVA__ format
2. Extract `.vmdk` disk images from the `.ova` files
    1. `tar -xvf image_name.ova`
        1. Replace `image_name` with the downloaded VM name
3. Convert the images into `qcow2` format
    1. `qemu-img convert -c -O qcow2 original_name.vmdk new_name.qcow2`
        1. Replace `original_name` with the extracted `.vmdk` name and `new_name` with something memorable
        2. The `-c` option enables compression making the resulting image __MUCH__ smaller
        3. The `-O qcow2` argument sets the output format
4. After saving the `.qcow2` images you can feel free to delete the other files
    1. It might be useful to keep everything until you get the VM running in case any errors happen

## Creating an __emulated__ machine in UTM

---

Do this whole process for both of the VM images

1. Open UTM
2. New Virtual Machine via __Emulate__
3. Choose __Other__ operating system
4. Check __skip ISO boot__ option and continue
5. Make sure the architecture is `x86_64` and use the default settings
    1. You can increase the memory to up to 40% of host memory (save some for qemu JIT cache)
    2. Let the CPU cores option stay as _default_ or empty
6. Specify a very small disk as this will be deleted later
7. Continue until you reach the final dialog displaying your configuration
    1. You can choose to share a folder with the VM. This is optional but helps with file transfer
8. Check __Show VM Settings__ and finish the setup
9. In the VM settings perform the following adjustments
    1. Go to the QEMU tab and un-check __UEFI Boot__
    2. On the left side of the VM settings find your __Drives__ and select the default one
    3. Delete the default disk
    4. Create a new disk by clicking __New Drive__ and select __Import...__ and find your `qcow2` file
    5. Make sure the new disk type is __Disk Image__ and interface __IDE__
10. Start the VM

#### Sharing files and clipboard to the VM

These require additional software that might not be present on the virtual machines

- SPICE guest agent `spice-vdagent`
- SPICE WebDAV `spice-webdavd`