# mtkHUmrftools

tools to decipher and compare MediaTek HeadUnit logo.mrf file

Note the shell script tools will download as .txt on macOS and Windows. Need to remove the .txt extension to use on UNIX/linux/macOS.

==== Versions

0.1	Initial proto dev release (macOS and Linux only, can use easy online free Linux VMs, directions below)  

==== Purpose

Just a couple of simple tools for comparing and dump a few parameters in MRF. Can be useful to check

- Are 2 MRFs same?  
- Extracted MRF have valid MRF bits? To confirm no extraction mistakes  

==== Programming Background

Initial version done by rusty UNIX shell programmer ( 40 years ago! )  

- Done on macOS  
- Require basic ~UNIX/Linux/macOS shell programming background  
- bash shell script might look simple but if don’t know what $PATH, ./, chmod +x, man are, then need basic learning on UNIX shell scripts to understand this stuff  

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
Identical in the 3 different MRFs I checked

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
If tools becomes useful to help HU owners to dump and compare extracted logo.mrf, PC version is desirable. Simple alternative now is use simple online VM Linux (see below)

====================================================================================
==== simple online free Linux VM to dump logo.bin
====================================================================================

No registration or login required. Simple to use

- download 24kdiff and mrfdump from repository  
- need to strip the .txt extension (win11 https://www.windowsdigitals.com/how-to-change-or-remove-file-extension-in-windows-11/ )  
- goto https://bellard.org/jslinux/?ref=itsfoss.com ( faq https://bellard.org/jslinux/faq.html )  
- click Startup Link for Alpine Linux 3.12.0 (probaby any will work)  
- upload 24kdiff mrfdump and your logo files via upload icon on bottom left  
- below is remaining screen capture, click code on upper left to see with out markdown formatting
-   (ls is list, like DOS DIR, chmod +x make file executable, ./ is path to current directory. So to run executable in current directory type ./executable_name

video https://youtu.be/nlz5M-ja_OU

Loading...  
 
Welcome to JS/Linux (i586)  
 
Use 'vflogin username' to connect to your account.  
You can create a new account at https://vfsync.org/signup .  
Use 'export_file filename' to export a file to your computer.  
Imported files are written to the home directory.  
 
localhost:~# ls
bench.py    hello.c     hello.js    logo.bin    mrfdump     readme.txt
localhost:~# chmod +x mrfdump
localhost:~# ./mrfdump -l logo.bin
 
**** logo.bin
 
= MD1ROM Header
 
length hex 0x07a3c00
length MB  7.6
name       logo
 
= Panel
 
Lvds
1024
600
 
= LOGO
 
1024
600

307200+0 records in
307200+0 records out
localhost:~# export_file logo.bin.png

====================================================================================
==== Simple online free Linux VM to compare logo.bin with display drivers
==== 15min run time on free online Linux VM. Much faster on own PC ( macOS / Linux )
====================================================================================

video https://youtu.be/vqCk5WipFVo

localhost:~# chmod +x 24kdiff
localhost:~# unzip Display\ driver\ only.zip
localhost:~# mv 24kdiff Display\ driver\ only
localhost:~# mv logo.bin Display\ driver\ only
localhost:~# cd Display\ driver\ only
localhost:~/Display driver only# find ./ -type f -name "*.gz" -exec ./24kdiff logo.bin "{}" \;

shell session below ====

localhost:~# ls
24kdiff                  hello.c                  readme.txt
Display driver only.zip  hello.js
bench.py                 logo.bin
localhost:~# chmod +x 24kdiff
localhost:~# unzip Display\ driver\ only.zip
Archive:  Display driver only.zip
  inflating: Display driver only/gongban_YT7216C_XHS-v1.73.8_e251a41c_ota.tar.gz
  inflating: Display driver only/57_720x1920_WD123FHM30AA-D2-v1.73.8_a78b3924_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_v1.70_e834d203_ota.tar.gz
  inflating: Display driver only/gongban_57_logo-v1.73.8_9b3df880_ota.tar.gz
  inflating: Display driver only/gongban_57xx_1024x768_LG-v1.73.8_6c2ecf45_ota.tar.gz
  inflating: Display driver only/gongban_YT5760B_YT-v1.73.8_fed699ec_ota.tar.gz
  inflating: Display driver only/gongban_57xx_768x1024_mipi-v1.73.8_c57dd4d2_ota.tar.gz
  inflating: Display driver only/57_720x1920_K1238S6248-Q05-01-v1.73.8_f330f7a1_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_WG_TBD_-v7.3_7_8239d0b4_ota.tar.gz
  inflating: Display driver only/gongban_57xx_JC_768x1024-v1.73.8_bead0010_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_JMG_12.1logo-v1.73.8_7144dfda_ota.tar.gz
  inflating: Display driver only/57_1200x2000_YT104IHLXL002-A-v1.73.8_a271ef72_ota.tar.gz
  inflating: Display driver only/57_720x1280_WD101HBM30TE-U6-v1.73.8_aa03f960_ota.tar.gz
  inflating: Display driver only/gongban_57xx_1024X600MIPI_9_ips-v1.73.8_c4982822_ota.tar.gz
  inflating: Display driver only/57_720x1280_WD090HBM30TC-U3-v1.73.8_086a15dc_ota.tar.gz
  inflating: Display driver only/57_1200x2000_YT095IBLXL001-A-v1.73.8_ceac73a7_ota.tar.gz
  inflating: Display driver only/gongban_57_logo-v1.73.8_f30aa089_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_HK_7707N_-v7.3_7_006172d6_ota.tar.gz
  inflating: Display driver only/gongban_57_SH-v1.73.8_1b9496ff_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_YT_FT8201-v1.73.8_47e9451e_ota.tar.gz
  inflating: Display driver only/gongban_57_KL_jd9365_-v7.3_7_3d7b6308_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_Yt7705-v1.73.8_0d702102_ota.tar.gz
  inflating: Display driver only/gongban_57_CarAUDIO_YT9707-v1.73.7_9f818621_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_sat7705-v1.73_c31afe38_ota.tar.gz
  inflating: Display driver only/gongban_YT72XX_CYXY_9882Q-v1.73.8_31902e48_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_SN9365_nosize-v1.73.8_9a3414e1_ota.tar.gz
  inflating: Display driver only/gongban_57_YT_ER88577_10inch-v1.73.8_78ed27c6_ota.tar.gz
  inflating: Display driver only/57_1200x2000_YT115IHLXL001-A-v1.73.8_8e32f8e1_ota.tar.gz
  inflating: Display driver only/logo.mrf PARTIAL SUMMARY
  inflating: Display driver only/gongban_YT57xx_1280x480-v1.73_09316688_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_WXlogo_jd9165-v1.73.8_31dae5d0_ota.tar.gz
  inflating: Display driver only/gongban_YT57_Android_XHS7705-v1.73.7_45756046_ota.tar.gz
  inflating: Display driver only/gongban_57_KL_JD9165-v1.73.8_2ae9d781_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_XMH_-v7.3_7_fb74e102_ota.tar.gz
  inflating: Display driver only/gongban_57_logo_YT_7705N-v1.73.8_80de7672_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_wx_jd9165-v7.3_7_d3913884_ota.tar.gz
  inflating: Display driver only/gongban_57_YT_ER88577_9inch-v1.73.8_dba6af49_ota.tar.gz
  inflating: Display driver only/57_1200x2000_YT129IBLXL001-A-v1.73.8_ebe9721c_ota.tar.gz
  inflating: Display driver only/gongban_57xx_1024x600MIPI_10_ips-v1.73.8_9d3bc14c_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_logo-v1.73.8_9a3414e1_ota.tar.gz
  inflating: Display driver only/57_1200_2000_WD104FHM30TA-A0-v1.73.8_f0a6779c_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_XHS_JD9366_-v7.3_7_ab683051_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_sat9707-v1.73.8_77e06f22_ota.tar.gz
  inflating: Display driver only/57_1200x2000_WD9582HM30TA-B0-v1.73.8_51c4cf8b_ota.tar.gz
  inflating: Display driver only/gongban_YT5760B_runshi_jd9165-v1.73.8_31dae5d0_ota.tar.gz
inflating: Display driver only/gongban_57_logo-v1.73.8_93f812d7_ota.tar.gz
  inflating: Display driver only/gongban_YT57XX_MC_-v7.3_7_e854a3c6_ota.tar.gz
  inflating: Display driver only/gongban_57xx_xinlu-v1.73.8_1b28fa72_ota.tar.gz
  inflating: Display driver only/gongban_YT5760B_OFD-v1.73.8_e3d35b58_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_768x1024_c0c4c660_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_1024x600-v1.73.8_65419550_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_1024x768-v1.73.8_a2a5ca07_ota.tar.gz
  inflating: Display driver only/gongban_YT57_1280x720-v1.73.7_e65f89a5_ota.tar.gz
  inflating: Display driver only/gongban_YT57xx_ui1-800x480-v1.72_42e1e595_ota.tar.gz
  inflating: Display driver only/57_1200x2000_WD1152HMTB-v1.73.8_78532029_ota.tar.gz
localhost:~#
localhost:~# ls
24kdiff                  Display driver only      Display driver only.zip  bench.py                 hello.c                  hello.js                 logo.bin                 readme.txt
localhost:~# mv 24kdiff Display\ driver\ only
localhost:~# mv logo.bin Display\ driver\ only
localhost:~# cd Display\ driver\ only
localhost:~/Display driver only# ls
24kdiff                                                       gongban_57_logo-v1.73.8_9b3df880_ota.tar.gz                   gongban_YT57XX_wx_jd9165-v7.3_7_d3913884_ota.tar.gz
57_1200_2000_WD104FHM30TA-A0-v1.73.8_f0a6779c_ota.tar.gz      gongban_57_logo-v1.73.8_f30aa089_ota.tar.gz                   gongban_YT57_1280x720-v1.73.7_e65f89a5_ota.tar.gz
57_1200x2000_WD1152HMTB-v1.73.8_78532029_ota.tar.gz           gongban_57_logo_YT_7705N-v1.73.8_80de7672_ota.tar.gz          gongban_YT57_Android_XHS7705-v1.73.7_45756046_ota.tar.gz
57_1200x2000_WD9582HM30TA-B0-v1.73.8_51c4cf8b_ota.tar.gz      gongban_57xx_1024X600MIPI_9_ips-v1.73.8_c4982822_ota.tar.gz   gongban_YT57xx_1024x600-v1.73.8_65419550_ota.tar.gz
57_1200x2000_YT095IBLXL001-A-v1.73.8_ceac73a7_ota.tar.gz      gongban_57xx_1024x600MIPI_10_ips-v1.73.8_9d3bc14c_ota.tar.gz  gongban_YT57xx_1024x768-v1.73.8_a2a5ca07_ota.tar.gz
57_1200x2000_YT104IHLXL002-A-v1.73.8_a271ef72_ota.tar.gz      gongban_57xx_1024x768_LG-v1.73.8_6c2ecf45_ota.tar.gz          gongban_YT57xx_1280x480-v1.73_09316688_ota.tar.gz
57_1200x2000_YT115IHLXL001-A-v1.73.8_8e32f8e1_ota.tar.gz      gongban_57xx_768x1024_mipi-v1.73.8_c57dd4d2_ota.tar.gz        gongban_YT57xx_768x1024_c0c4c660_ota.tar.gz
57_1200x2000_YT129IBLXL001-A-v1.73.8_ebe9721c_ota.tar.gz      gongban_57xx_JC_768x1024-v1.73.8_bead0010_ota.tar.gz          gongban_YT57xx_JMG_12.1logo-v1.73.8_7144dfda_ota.tar.gz
57_720x1280_WD090HBM30TC-U3-v1.73.8_086a15dc_ota.tar.gz       gongban_57xx_xinlu-v1.73.8_1b28fa72_ota.tar.gz                gongban_YT57xx_WXlogo_jd9165-v1.73.8_31dae5d0_ota.tar.gz
57_720x1280_WD101HBM30TE-U6-v1.73.8_aa03f960_ota.tar.gz       gongban_YT5760B_OFD-v1.73.8_e3d35b58_ota.tar.gz               gongban_YT57xx_YT_FT8201-v1.73.8_47e9451e_ota.tar.gz
57_720x1920_K1238S6248-Q05-01-v1.73.8_f330f7a1_ota.tar.gz     gongban_YT5760B_YT-v1.73.8_fed699ec_ota.tar.gz                gongban_YT57xx_Yt7705-v1.73.8_0d702102_ota.tar.gz
57_720x1920_WD123FHM30AA-D2-v1.73.8_a78b3924_ota.tar.gz       gongban_YT5760B_runshi_jd9165-v1.73.8_31dae5d0_ota.tar.gz     gongban_YT57xx_logo-v1.73.8_9a3414e1_ota.tar.gz
gongban_57_CarAUDIO_YT9707-v1.73.7_9f818621_ota.tar.gz        gongban_YT57XX_HK_7707N_-v7.3_7_006172d6_ota.tar.gz           gongban_YT57xx_sat7705-v1.73_c31afe38_ota.tar.gz
gongban_57_KL_JD9165-v1.73.8_2ae9d781_ota.tar.gz              gongban_YT57XX_MC_-v7.3_7_e854a3c6_ota.tar.gz                 gongban_YT57xx_sat9707-v1.73.8_77e06f22_ota.tar.gz
gongban_57_KL_jd9365_-v7.3_7_3d7b6308_ota.tar.gz              gongban_YT57XX_SN9365_nosize-v1.73.8_9a3414e1_ota.tar.gz      gongban_YT57xx_ui1-800x480-v1.72_42e1e595_ota.tar.gz
gongban_57_SH-v1.73.8_1b9496ff_ota.tar.gz                     gongban_YT57XX_WG_TBD_-v7.3_7_8239d0b4_ota.tar.gz             gongban_YT7216C_XHS-v1.73.8_e251a41c_ota.tar.gz
gongban_57_YT_ER88577_10inch-v1.73.8_78ed27c6_ota.tar.gz      gongban_YT57XX_XHS_JD9366_-v7.3_7_ab683051_ota.tar.gz         gongban_YT72XX_CYXY_9882Q-v1.73.8_31902e48_ota.tar.gz
gongban_57_YT_ER88577_9inch-v1.73.8_dba6af49_ota.tar.gz       gongban_YT57XX_XMH_-v7.3_7_fb74e102_ota.tar.gz                logo.bin
gongban_57_logo-v1.73.8_93f812d7_ota.tar.gz                   gongban_YT57XX_v1.70_e834d203_ota.tar.gz                      logo.mrf PARTIAL SUMMARY
localhost:~/Display driver only#
localhost:~/Display driver only# find ./ -type f -name "*.gz" -exec ./24kdiff logo.bin "{}" \;
Files logo.bin.24k and ./gongban_YT7216C_XHS-v1.73.8_e251a41c_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_720x1920_WD123FHM30AA-D2-v1.73.8_a78b3924_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_v1.70_e834d203_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_logo-v1.73.8_9b3df880_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_1024x768_LG-v1.73.8_6c2ecf45_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT5760B_YT-v1.73.8_fed699ec_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_768x1024_mipi-v1.73.8_c57dd4d2_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_720x1920_K1238S6248-Q05-01-v1.73.8_f330f7a1_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_WG_TBD_-v7.3_7_8239d0b4_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_JC_768x1024-v1.73.8_bead0010_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_JMG_12.1logo-v1.73.8_7144dfda_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_YT104IHLXL002-A-v1.73.8_a271ef72_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_720x1280_WD101HBM30TE-U6-v1.73.8_aa03f960_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_1024X600MIPI_9_ips-v1.73.8_c4982822_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_720x1280_WD090HBM30TC-U3-v1.73.8_086a15dc_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_YT095IBLXL001-A-v1.73.8_ceac73a7_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_logo-v1.73.8_f30aa089_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_HK_7707N_-v7.3_7_006172d6_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_SH-v1.73.8_1b9496ff_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_YT_FT8201-v1.73.8_47e9451e_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_KL_jd9365_-v7.3_7_3d7b6308_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_Yt7705-v1.73.8_0d702102_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_CarAUDIO_YT9707-v1.73.7_9f818621_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_sat7705-v1.73_c31afe38_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT72XX_CYXY_9882Q-v1.73.8_31902e48_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_SN9365_nosize-v1.73.8_9a3414e1_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_YT_ER88577_10inch-v1.73.8_78ed27c6_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_YT115IHLXL001-A-v1.73.8_8e32f8e1_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_1280x480-v1.73_09316688_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_WXlogo_jd9165-v1.73.8_31dae5d0_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57_Android_XHS7705-v1.73.7_45756046_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_KL_JD9165-v1.73.8_2ae9d781_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_XMH_-v7.3_7_fb74e102_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_logo_YT_7705N-v1.73.8_80de7672_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_wx_jd9165-v7.3_7_d3913884_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_YT_ER88577_9inch-v1.73.8_dba6af49_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_YT129IBLXL001-A-v1.73.8_ebe9721c_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_1024x600MIPI_10_ips-v1.73.8_9d3bc14c_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_logo-v1.73.8_9a3414e1_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200_2000_WD104FHM30TA-A0-v1.73.8_f0a6779c_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_XHS_JD9366_-v7.3_7_ab683051_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_sat9707-v1.73.8_77e06f22_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_WD9582HM30TA-B0-v1.73.8_51c4cf8b_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT5760B_runshi_jd9165-v1.73.8_31dae5d0_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57_logo-v1.73.8_93f812d7_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57XX_MC_-v7.3_7_e854a3c6_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_57xx_xinlu-v1.73.8_1b28fa72_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT5760B_OFD-v1.73.8_e3d35b58_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_768x1024_c0c4c660_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_1024x600-v1.73.8_65419550_ota.tar.gz.24k are identical
Files logo.bin.24k and ./gongban_YT57xx_1024x768-v1.73.8_a2a5ca07_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57_1280x720-v1.73.7_e65f89a5_ota.tar.gz.24k differ
Files logo.bin.24k and ./gongban_YT57xx_ui1-800x480-v1.72_42e1e595_ota.tar.gz.24k differ
Files logo.bin.24k and ./57_1200x2000_WD1152HMTB-v1.73.8_78532029_ota.tar.gz.24k differ
localhost:~/Display driver only# 

