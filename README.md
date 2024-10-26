# mtkHUmrftools

tools to decipher and compare MediaTek HeadUnit logo.mrf file

==== Versions

0.1	Initial proto dev release (macOS and Linux only, can use easy online free Linux VMs, directions below)  

==== Purpose

Just a couple of simple tools for comparing and dump a few parameters in MRF. Can be useful to check

- Are 2 MRF same?  
- Extracted MRF have valid MRF bits? To confirm no extraction mistakes  

==== Programming Background

Initial version done by rusty UNIX shell programmer ( 40 years ago! )  

- Done on macOS  
- Require basic ~UNIX/Linux/macOS shell programming background  
- She’ll script might look simple but if don’t know what $PATH, ./, chmod +x, man are, then need basic learning on UNIX shell scripts to understand this stuff  

==== Architecture Understanding

MRF stands for MediaTek ROM Format

logo.mrf contains boot logo (user can change later using HU software). This logo displays immediately on cold boot. Naturally this require setting up the LCD Panel and send it the boot logo image. Logo.mrf contains panel setup parameters. Likely lvds and mipi commands,

LCD Panel model supply is likely constantly evolving (new and cheaper model). HU “integrator” likely is responsible to combine parts from Panel supplier with board supplier. As panels change, logo.mrf need to match. And likely will mismatch other panel types.

These Android HUs appears to be somewhat customized Android for HU application. The display driver architecture is likely “distributed” with parts done at boot for displaying logo, and parts done later when Android is booted.

Therefore, flashing unknown logo.mrf can result in bricking the display.

==== MRF Format

Need to reverse engineer this format

Tools Used

binxelview to inspect boot logo picture ( https://github.com/bbbradsmith/binxelview )  
Binary Ninja to inspect hex and file header ( https://binary.ninja/free/ )  

0 - 1FFh

This is the MD1ROM header ( use Binary Ninja Linear view ). Contains  
Total valid logo.mrf  length  
File name  
Next block offset (always 200h?)  
Padded with FFh to end of this region  

200h - 2FFh

MRFv1.1.1  
Likely the remaining format spec. Maybe some offsets to other regions?  

300h - 4FFh

Panel Section  
Contains 2 resolution #s, doesn't help with orientation from examining fiew mrfs  

500h - DFFh

LOGO description section  
Contains 2 resolution #s, doesn't help with orientation from examining fiew mrfs  

E00h - ??? ( < 5E00h )  

Panel Setup Parameters  
couldn't decode mipi data against mipi command protocol  
	2009 mipi DCS v1.0.2 ( link )  
	no access to latest mipi DCS v2.1 ( link, need to be mipi alliance member company )  
	lots of bytes are 0xff, ascii { } , . and space (0x20) these are likely the usual typical separators  

5E00h - ~8MB

multiple lengths exist  
end address = header length - 1  

5E00h - 7A3DFFh	1024*600 lvds horizontal/vertical, 7705N 1280*720 mipi  
5E00h - 805DFFh	9365 1280*720 mipi (larger due to logo image extending beyond other logo.mrf length)  

2 copies of boot logo image is here  
User changing boot logo changed only first copy on my 1024*600 HU  
Pixel format appears to be 32bit RGBA (or BGRA, didn’t check RGB order). A=0xFF  
Blocks of 0s between 2 images  
0 padding after 2nd image to the end  

binxelview settings to view images

Byte position to 5e00 to skip the first ~24k  
BPP = 32 (play with 2 parameter box below 4,4 or 4,0)  
Width Try common width and height to see image 1280, 1024, 720, 768, 800 etc.  
Height = 1  

GUESSES

- Likely the first slightly less then 24K is enough to program panel properly
- This probably can be considered as “signature” of this “panel driver” Thus, develop tools to compare this region. Probably no need to compare logo images which can be different if extracted from HU and user changed boot logo

==== Tools

See mrfdump, 24kdiff

==== TODOS

Find HU orientation info would be hugely helpful
Find HU LCD panel type would be ideal but probably difficult without knowning manufacturers code for the panel type. Doesn't have panel model string.
If tools becomes useful to help HU owners to check and compare extracted logo.mrf, PC version is desirable. Simple alternative now is use simple online VM Linux (see below)

==== simple VM Linux

No registration or login required. Simple to use

- download 24kdiff and mrfdump from repository  
- need to strip the .txt extension (win11 https://www.windowsdigitals.com/how-to-change-or-remove-file-extension-in-windows-11/ )  
- goto https://bellard.org/jslinux/?ref=itsfoss.com ( faq https://bellard.org/jslinux/faq.html )  
- click Startup Link for Alpine Linux 3.12.0 (probaby any will work)  
- upload 24kdiff mrfdump and your logo files via upload icon on bottom left  
- below is remaining screen capture, click code on upper left to see with out markdown formatting
-   (ls is list, like DOS DIR, chmod +x make file executable, ./ is path to current directory. So to run executable in current directory type ./executable_name

Loading...  
 
Welcome to JS/Linux (i586)  
 
Use 'vflogin username' to connect to your account.  
You can create a new account at https://vfsync.org/signup .  
Use 'export_file filename' to export a file to your computer.  
Imported files are written to the home directory.  
 
localhost:~# ls  
24kdiff     hello.c     hlogo.mrf   readme.txt  
bench.py    hello.js    mrfdump     rhlogo.mrf  
localhost:~# chmod +x 24kdiff  
localhost:~# chmod +x mrfdump  
localhost:~# ./24kdiff hlogo.mrf rhlogo.mrf  
comparing hlogo.mrf rhlogo.mrf  
Files hlogo.mrf.24k and rhlogo.mrf.24k are identical  
localhost:~# ./mrfdump hlogo.mrf  
 
 
==== hlogo.mrf  
 
= MD1ROM Header  
 
partition length 0x07a3c00  
partition name logo  
 
= Panel  
 
Lvds  
1024  
600  
 
= LOGO  
 
1024  
600  
