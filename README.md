# volatility2 profiles

This is for future reference - the process I took to create volatility profiles for an assignment, in this case for **Debian**.  
🚮 _(it was probably a pretty dump approach, but maybe it helps someone.)_  

## Process

Identify Target Profile:
```
$ vol.py -f /path/to/your/memory.dump linuxgetprofile
```

Setup VM with target OS.   
Install `dwarfdump` and `build-essential` (if your target OS is Debian).  
(You can reference the [Wiki](https://github.com/volatilityfoundation/volatility/wiki/Linux#creating-a-new-profile) for another OS).  

```
$ su
# apt update
# apt install linux-image-<target-kernel-version>
# apt install linux-image-686-pae
# apt install dwarfdump
# apt install build-essential
```

Then download Volatility 2 (https://github.com/volatilityfoundation/volatility), via `git` or otherwise.  
Restart the system and boot with new kernel.  
  
Normaly you could download the kernel headers from the Debian repository.  
```
# apt install kernel-headers-$(uname -r)
```  
But this is where the extra steps come in:  
Since the kernel headers were missing from the Debian repository, I had to generate them otherwise. Here I used `module-assistant`:  

``` 
$ su
# apt install module-assistant
# m-a prepare
# ls /usr/src
[...] linux-OLDVERSION.1655303173
```
The makefile in volatility expects a directory `/build/` under `/lib/modules/<target-kernel-version>`, so I created a symlink.
``` 
# cd /lib/modules/4.19.0-12-686
# ln -s /usr/src/linux-OLDVERSION.1655303173 build
```
Then create `module.dwarf`.
```
# cd /path/to/volatility/tools/linux/
# make
# ls
[...] module.dwarf
```
You will find the `System.map-<kernel-version>` file under `/boot/`.  
(Or refer to the [Wiki](https://github.com/volatilityfoundation/volatility/wiki/Linux#creating-a-new-profile) again if another approach may apply.)

Then you can zip `System.map-<kernel-version>` and `module.dwarf` and the profile is ready to go.
```
$ zip ./<profile-name> volatility/tools/linux/module.dwarf /boot/System.map-<kernel-version>
```
Copy the profile to `volatility/plugins/overlays/linux/`.


## ISO Archives

https://cdimage.debian.org/cdimage/archive/