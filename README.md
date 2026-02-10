# arch-linux-installation
My arch linux install process

## Step 1: Install Arch Linux ISO and verify its.
Download link: [Arch Linux](https://archlinux.org/download/) <br>
    - Install .iso and .iso.sig file <br>
    * Verify with GnuPG (remember to install GnuPG) <br>
    + The download the signature and verify it <br>
```
gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org

gpg --verify archlinux-2026.02.01-x86_64.iso.sig archlinux-2026.02.01-x86_64.iso
```

    - If it is good signature then you can start the next process.

## Step 2: Install Ventoy.
Download Link: [Ventoy](https://www.ventoy.net/en/download.html) <br>
    - After download ventoy, run it download file and install ventoy to USB. <br>
    * Then, move .iso file to the USB.

## Step 3: Download Arch Linux

