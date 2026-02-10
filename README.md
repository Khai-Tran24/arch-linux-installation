# arch-linux-installation
My arch linux install process

## Step 1: Install Arch Linux ISO and verify its.
Download link: https://archlinux.org/download/
    - Install .iso and .iso.sig file
    - Verify with GnuPG (remember to install GnuPG)
    - The download the signature. Parsing the below command to cmd
```bash
     gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org
```
    - Then, verify the signature
    ```bash 
        gpg --verify archlinux-2026.02.01-x86_64.iso.sig archlinux-2026.02.01-x86_64.iso
    ```    
    - If it is good signature then you can start the next process.

## Step 2: Install Ventoy.
Download Link: https://www.ventoy.net/en/download.html
    - After download ventoy, run it download file and install ventoy to USB.
    - Then, move .iso file to the USB.

## Step 3: Download Arch Linux

