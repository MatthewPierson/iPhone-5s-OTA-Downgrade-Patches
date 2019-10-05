# iPhone 5s OTA Downgrade Pateches
 Patches for downgrading an iPhone 5s to 10.3.3 with OTA blobs.

iBSS/iBEC Patches are only for iPhone6,2 and iPhone6,1 - NON OTHER AT THIS STAGE.

Futurerestore patch is for V246/V245, the latest version and latest release.

This guide assumes you have the latest liboffsetfinder64, iBoot64patcher, img4tool, img4lib, irecovery, tsschecker, bspatch, python and all the dependencies installed and updated to the latest version. I'm not going to help you install/compile these programs because I don't have time to help everyone sadly. It should be straight forward to compile and install everything, just google things and read errors if you get them. 
----------------------------------------------------------------------           
If this is shit or doesn't make sense I'm sorry, I wrote this at 3am and on 3 hours of sleep :) 
----------------------------------------------------------------------           

Note: If you don't want to patch iBSS/iBEC yourself or can't compile any of the programs then I have provided .patch files below. Please read the whole post though, so you don't miss anything.

----------------------------------------------------------------------

COMPATIBILITY: At the moment only the iPhone 5s (s5l8960x) is supported. I will create more patch files when Linus updates his rmsigchks.py for more A7 devices. 
----------------------------------------------------------------------
Note that this IS an untethered downgrade as we are using OTA blobs meaning that the install of iOS is signed and won't need to be booted from pwndfu mode everytime unless you are booting in verbose mode.
----------------------------------------------------------------------
~~Currently only the iPhone6,2 has patch files as this is the 5s that I have. If requested I can create patch files for the iPhone6,1 but you can do those yourself if you want to.~~ Turns out I'm stupid and 6,1 shares iBSS/iBEC with 6,2. Have uploaded new patches to fix another issue but if someone with a 6,1 can test that'd be great.
----------------------------------------------------------------------
I am planning on updating this guide soon to show how to boot in verbose mode. The way I use currently isn't amazing so I want to figure that out before I post how to.
----------------------------------------------------------------------

----------------------------------------------------------------------
First download the 10.3.3 ipsw from [here](https://ipsw.me/). Extract the contents of said ipsw and traverse from the root directory to /Firmware/dfu/ and grab iBSS.iphone6.RELEASE.im4p and iBEC.iphone6.RELEASE.im4p

Move the two files into a folder with iBoot64patcher, img4tool and img4lib (img4 is name of binary for img4lib, and yes img4tool and img4 are very different you need both).

Go to https://www.theiphonewiki.com/wiki/Firmware_Keys/10.x and click the link for the keys for 10.3.3 for your device

Find the IV and Key for iBSS and iBEC.

Put the two numbers together as one with the IV before the Key so for iphone6,2 iBSS the IV is 

    f2aa35f6e27c409fd57e9b711f416cfe 

and the Key is 

    599d9b18bc51d93f2385fa4e83539a2eec955fce5f4ae960b252583fcbebfe75 

so the final number is 

    f2aa35f6e27c409fd57e9b711f416cfe599d9b18bc51d93f2385fa4e83539a2eec955fce5f4ae960b252583fcbebfe75

Now you need to decrypt iBSS and iBEC 

    ./img4 -i iBSS.iphone6.RELEASE.im4p -o ibss.decrypt -k “ivkey” -D” 

same command for iBEC just with file names and different ivkey. 

MAKE SURE TO INCLUDE "-D" OTHERWISE IT WON'T DECRYPT THE IMAGE 
----------------------------------------------------------------------
----------------------------------------------------------------------

Next run img4tool to extract the raw binary from the decrypted images as iboot64patcher does not support im4p and img4 files at the moment. 

Run 

    ./img4tool -e -o ibss.raw ibss.decrypt 

Same for iBEC, just change file names.

----------------------------------------------------------------------

Now you need to run iBoot64patcher. Here you can choose the boot-args you want to use, e.g here is where you enable verbose boot.

     ./iBoot64patcher ibss.raw ibss.pwn


    ./iBoot64patcher ibec.raw ibec.pwn -b “add-your-boot-args-here”


As far as I know, you don’t pass boot args to iBSS but I might be wrong. If you aren't sure then just use my verbose patch files to get verbose boot to work as I know they work.

----------------------------------------------------------------------

Next, use img4tool to do some cool shit.


     ./img4tool -p ibss.im4p --tag ibss --info iBoot-hax ibss.pwn
    
    ./img4tool -p ibec.im4p --tag ibec --info iBoot-hax ibec.pwn

----------------------------------------------------------------------

Now you need to use img4tool again but with some shsh. Lets get the shsh for 10.3.3 ota first.

Download and install the latest tsschecker if you don’t have it already. Then run 

    ./tsschecker -e “your-ecid” -s -o -i 9.9.10.3.3 --buildid 14G60 -d iPhone6,2(or whatever your device is) --save-path “/where/futurerestore/is” 

 This will save shsh for your device for 10.3.3 to where you specified . 

----------------------------------------------------------------------
Now use img4tool as follows 

    ./img4tool -p ibss.im4p -c ibss.img4 -s “/path/to/shsh/you/saved/” 

    ./img4tool -p ibec.im4p -c ibec.img4 -s “/path/to/shsh/you/saved/” 

Now you have patched iBSS and iBEC that you can use to downgrade!

----------------------------------------------------------------------

Now, for those who don’t want to mess around with that, I’ll be providing patch files for iBSS/iBEC that you can use. You can download all the .patch files from my [github repo](https://github.com/MatthewPierson/iPhone-5s-OTA-Downgrade-Patches)

First make sure you have "bspatch" installed then get the stock iBSS and iBEC from the 10.3.3 ipsw and place them in a folder with the .patch files.

Now if you want verbose then run 

    bspatch iBSS.iphone6.RELEASE.im4p ibss.patched ibss.verbose.patch

If you don’t then run 

    bspatch iBSS.iphone6.RELEASE.im4p ibss.patched ibss.normal.patch

Now do the same for iBEC. I have since added more patches, use i***.verbose.restore.patch to use verbose mode while restoring, i***.verbose.patch to boot tethered verbose mode (will add guide soon) or use i***.normal.patch to just patch normally without verbose.

Note: I found that for switching from pwndfu to pwnrecovery later on only the verbose iBSS and iBEC worked so if irecovery fails or stops when sending iBEC then trying using the verbose files instead.
----------------------------------------------------------------------
----------------------------------------------------------------------
 
Now you need a modified version of futurerestore (currently, tihmstar is updating the official version but for now we have to make do). 

I used s0uthwest’s fork at latest version, 246, and modified it. You will need to download the latest release (245) and apply this patch to the futurerestore binary. You can also git clone the latest version, 246, and build from source then patch but either works I have tested both.

    bspatch futurerestore futurerestore_patched futurerestore.patch

Now delete the old fututrerestore binary file and rename the new patched one to “futurerestore”

----------------------------------------------------------------------

Now download/clone Linus’s fork of ipwndfu from [here](https://github.com/LinusHenze/ipwndfu_public). cd into the ipwndfu_public folder and put your device into dfu mode then connect it to your macos device (hackintosh or legit mac, either is fine). 

Run 

    ./ipwndfu -p

to get into pwndfu mode. Now this will fail a lot of times as that is just the nature of this exploit on the A7. That’s expected just keep trying. I found closing itunes and iTunesHelper to help a bit but results may vary. 

----------------------------------------------------------------------

Once in pwndfu mode, run 

    python rmsigchks.py

and if all goes well it should return with 

    "Device is now ready to accept unsigned images"

----------------------------------------------------------------------

Now download the latest irecovery. Once done, you need to send a random dummy file to the device. This can be anything but I use a small .txt file. Run 

    ./irecovery -f random.txt

After that runs and the device reconnects, you can send your pwned ibss and ibec =).

    ./irecovery -f ibss.img4

 Then once that sends and device reconnects run 

    ./irecovery -f ibec.img4

and you will be able to futurerestore to 10.3.3 as you are now in pwnrecovery!

Also download the 10.3.3 OTA build manifest from Alitek. Linked [here](https://twitter.com/alitek123/status/924877863576330240)
----------------------------------------------------------------------
----------------------------------------------------------------------

Now we need to edit the stock 10.3.3 ipsw that we downloaded at the start. For this you will need a program that can edit the contents of a zip without breaking it. On windows I used 7Zip to do this, not sure what you can use for macOS but I know that there is programs that can do this. Easiest way to do use 7Zip on windows however.

You need to grab the pwned iBSS and iBEC that you created before and rename them to match the original names that they had inside the ipsw. iBSS needs to be named iBSS.iphone6.RELEASE.im4p and iBEC needs to be named iBEC.iphone6.RELEASE.im4p. Now overwrite the current iBSS and iBEC inside the ipsw and once it repacks and is complete you have a custom ipsw to dowgrade with!

----------------------------------------------------------------------

Now the shsh you downloaded will not match the current apnonce of the device. ~~My way of getting around this is attempting a restore with the mismatched shsh, finding the current apnonce of the device,~~ Use igetnonce to get the apnonce of the device and grab shsh with the current apnonce of the device (Credit to [rA9](https://twitter.com/BarisUlasCukur) for reminidng me that igetnonce is a thing). Run 

    ./igetnonce

It will print out the apnonce for the device.

Now use this apnonce and request a new ticket. 

Run 

    ./tsschecker -e “your-ecid” -s -o -i 9.9.10.3.3 --buildid 14G60 -d iPhone6,2(or whatever your device is) --save-path “/where/futurerestore/is” --apnonce “the number we just grabbed” 

This will grab shsh with the correct apnonce that your device currently has!

Now run futurerestore again but with the new shsh 

    ./futurerestore -t “new-shsh-file” -b baseband from 10.3.3 ipsw -p Alitek's_OTA_buildmanifest.plist -s sep from 10.3.3 ipsw -m Alitek's_OTA_buildmanifest.plist 10.3.3.ipsw


Phone should now restore to 10.3.3 with no issues! Make sure you have a good amount of storage availible when futurerestoreing, I ran into an issue where the restore failed because I ran out of SSD space.

----------------------------------------------------------------------

If you run into any issues, which I expect as this guide/tutorial probably contains some errors, just feel free to either comment here or dm me on [twitter](https://twitter.com/mosk_i). Though i'm more likely to reply here because twitter sucks.


Credits go to: [axi0mx](https://twitter.com/axi0mX) (checkm8), [Tihmstar](https://twitter.com/tihmstar) (img4tool, futurerestore, iBoot64patcher, liboffsetfinder64 and probably more), [Linus](https://twitter.com/LinusHenze) (ipwndfu fork with removedsigpatches), [alitek12](https://twitter.com/alitek12) (OTA Buildmanifest for A7 devices), [xerub](https://twitter.com/xerub ) (img4lib) and [S0uthwes](https://twitter.com/s0uthwes)(futurerestore fork).
----------------------------------------------------------------------