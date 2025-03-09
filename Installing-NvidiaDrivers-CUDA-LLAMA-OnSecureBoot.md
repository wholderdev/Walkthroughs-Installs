# Installing CUDA and NVIDIA drivers for a Secure Boot environment, to setup llama.cpp LLM tools

This is an example of how I installed NVIDIA drivers for my debian machine dual booted with windows 11, which requires Secure Boot.



## Setting up your environment

Give sudo access to user

```
# Make sure to use su - and not su root, as the latter doesn't provide us the usermod binary
su -
usermod -aG sudo myUser
exit
```

Log out then log back in



Check and make sure secure boot is actually enabled

```
sudo mokutil --sb-state
#	Expected output: SecureBoot enabled
```

If it's not enabled, I recommend you don't bother with the secure boot steps with creating a key and registering it.



(preference) Install tmux and vim

```
sudo apt install tmux vim
```



Add contrib and non-free to /etc/apt/sources.list

```
sudo vim /etc/apt/sources.list

# In vim
#   (i to insert text)
:%s/main/main contrib non-free/g
#   (ESC to stop inserting, :wq to exit)

# update apt
sudo apt update
```


Download the kernel headers for your system just in case the nvidia compile needs it.

```
sudo apt install linux-headers-$(uname -r)
```


## Creating and setting up secure boot keys

Secure boot key generation

```
# Once again, use root
su -

# Probably not necessary, but just to make sure we're in /root/
cd ~

# Generate the Keys
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.key -out MOK.crt -nodes -days 3650 -subj "/CN=Secure Boot Signing Key/"

# Convert the format to DER
openssl x509 -in MOK.crt -out MOK.cer -outform DER

exit
```

*The MOK.key is a sensitive key, make sure only root can access it.*



Now import the MOK.cer key to register it for the Secure Boot's Machine Owner Key. Provide a password that will be typed after the reboot.

```
sudo mokutil --import /root/MOK.cer
# Type a password (you are creating one, write it down)
```


Check to make sure the key was imported, this makes sure the promp will appear next boot

```
sudo mokutil --list-new
# Just make sure something came up, check the Issuer and make sure it says "CN=Secure Boot Signing Key".
```


Now reboot (be ready to press  a key to go to MOK management)

```
sudo reboot
```



Now on reboot, hit any key and go to the MOK management page.

Select Enroll MOK

Feel free to view the key and check for “CN=Secure Boot Signing Key”

Hit Continue, then hit Yes

Type the password you wrote down for the key

Hit Reboot

Feel free to check in dmesg if the cert shows up and was loaded

```
sudo dmesg | grep "Secure Boot Signing Key"
```



Install dkms

```
sudo apt install dkms
```



I added the following lines to /etc/dkms/framework.conf

```
mok_signing_key="/root/MOK.key"
mok_certificate="/root/MOK.crt"
```



Reconfigure DKMS

```
sudo dpkg-reconfigure dkms
```



Installing CUDA and NVIDIA drivers

Go through the following install process. Here's a link to the nvidia instructions: https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Debian&target_version=12&target_type=deb_local

Here are the commands I ran

```
wget https://developer.download.nvidia.com/compute/cuda/12.8.1/local_installers/cuda-repo-debian12-12-8-local_12.8.1-570.124.06-1_amd64.deb
sudo dpkg -i cuda-repo-debian12-12-8-local_12.8.1-570.124.06-1_amd64.deb
sudo cp /var/cuda-repo-debian12-12-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-8
sudo apt-get install -y nvidia-open

sudo reboot
```



Check if it detects the graphics card now

```
sudo nvidia-smi
```


Give it a config file

```
sudo nvidia-xconfig
```

Now that the cuda drivers are installed, let's add it to our path by creating a script at /etc/profile.d/cuda.sh

```
sudo vim /etc/profile.d/cuda.sh

# Add the following lines
#   (i to insert text)
export PATH=$PATH:/usr/local/cuda/bin

# Set permissions to rw for root, and r for everyone else
sudo chmod 644 /etc/profile.d/cuda.sh

# Source the script
source /etc/profile.d/cuda.sh

#   (ESC to stop inserting, :wq 
```

Feel free to add the export to your path at .profile or .bashrc instead, if it isn't sourcing the script on boot.

## Installing and testing llama.cpp

Now do a git clone of the llama.cpp project

```
cd ~
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
```

Now run the following cmake commands. If cmake is not installed, install it.

```
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```

If the build is successful, add the bin directory to your path.

```
export PATH=$PATH:/home/myUser/llama.cpp/build/bin
```

If you'd like to add it the same way the cuda drivers were added

```
sudo vim /etc/profile.d/llama.sh

# Add the following lines
#   (i to insert text)
export PATH=$PATH:/home/myUser/llama.cpp/build/bin

# Set permissions to rw for root, and r for everyone else
sudo chmod 644 /etc/profile.d/llama.sh

# Source the script
source /etc/profile.d/llama.sh

#   (ESC to stop inserting, :wq to exit)
```

Now lets download an example and test if it works. Refer to https://huggingface.co for gguf files to test

```
cd ~
mkdir gguf-files
cd gguf-files
wget https://huggingface.co/TheBloke/tinyllama-1.1b-chat-GGUF/resolve/main/tinyllama-1.1b-chat.Q4_K_M.gguf
llama-cli -m tinyllama-1.1b-chat.Q4_K_M.gguf

# Feel free to change the gpu layers or add other tags
llama-cli -m tinyllama-1.1b-chat.Q4_K_M.gguf --gpu-layers 60 --interactive
```


If I run into any issues, or if anyone stumbles upon this and needs troubleshooting, I can see if any instructions need updates or are incorrect.



Sources

https://wiki.debian.org/SecureBoot#Using_your_key_to_sign_modules

https://wiki.debian.org/NvidiaGraphicsDrivers/Troubleshooting

https://www.rodsbooks.com/efi-bootloaders/secureboot.html#signing

https://www.rodsbooks.com/efi-bootloaders/secureboot.html#really-signing

https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Debian&target_version=12&target_type=deb_local

https://www.reddit.com/r/linux4noobs/comments/18n34c3/nvidia_drivers_for_debian_12_step_by_step/

https://github.com/ggml-org/llama.cpp/tree/master

https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md



# Troubleshooting

Here are some issues with solutions, though I don't think you need to follow these.

## Blacklisting Nouveau

Create the /etc/modprobe.d/blacklist-nouveau.conf file (may be redundant in the /etc/modprobe.d/nvidia.conf file)

```
sudo vim /etc/modprobe.d/blacklist-nouveau.conf

# Write the following
#   (i to insert text)
blacklist nouveau
options nouveau modeset=0
#   (ESC to stop inserting, :wq to exit)
```


## Updating initramfs

The install process should do this for you, but feel free to make sure the drivers are properly loaded on boot.

```
sudo update-initramfs -u
sudo reboot
```


## Getting rid of the ACPI Errors

For whatever reason I keep getting these errors. I don't know what's causing them. Maybe it's a reading comprehension issue from me reading the logs. Here's how to force ACPI to be enabled and to stop the errors from logging.

Note: I actually removed this from my install, I was troubleshooting the "No login screen" issue below and thought ACPI was to blame initially.

```
sudo vim /etc/default/grub

# Add the following to GRUB_CMDLINE_LINUX_DEFAULT
#   (i to insert text)
acpi=force pci=noaer

# Should look similar to this
GRUB_CMDLINE_LINUX_DEFAULT="quiet acpi=force pci=noaer"
#   (ESC to stop inserting, :wq to exit)

# Run the following to update grub
sudo update-grub

# Reboot
sudo reboot
```


## No login screen (my own goof)

Make sure there's not a third monitor that's not on or is being used by another computer. I was sharing a monitor with another computer, and the login screen was showing up on that monitor while it was occupied.
