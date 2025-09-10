# Ultimate-HyperHDR-Ambilight-fine-tuning-experience-for-LG-webOS-with-new-LUT-calibration-

# Attention!

This guide is not part of the official HyperHDR for WebOS from the homebrew channel and should not be followed unless you are willing to try something different from what is provided in the homebrew channel and get the maximum colour rendering from different LUTs. 
Everything here is experimental and constantly being tweaked as @awawa-dev makes changes.
There is no guarantee that the colour reproduction will be equally good on all devices.
You may need to make adjustments yourself or even calibrate your external playback source. 
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Note
Since some inexperienced users recently found this guide too complicated and also encountered errors when running it, I decided to simplify it so that everyone can run it.

With the changes in the PicCap UI, the additional code from @neogeo for the HyperHDR installer and the countless tips from @awawa-dev, I was finally able to put together a new HyperHDR version that simplifies everything.

This guide is based on @awawa-dev NV12 implementation and new LUT calibration method in HyperHDR, @sundermann NV12 customisation in hyperion-webos, @neogeo Add Settings Panel with LUT installation selector option in Settings menu of hyperhdr-webos-loader and Add NV12 to PicCap-UI by @TBSniller & @satgit62.

PiCap v0.5.1 and HyperHDR 21.0.0.0 (org.webosbrew.hyperhdr.loader_0.2.5_all.ipk) are required to use this guide.

The implementation and utilization of the NV12 image format in HyperHDR now offers numerous enhancements and the capability to achieve precise color matching even in our exotic LG webOS by utilizing the YUV format in lieu of RGB.

# Advantages:

The NV12-YUV conversion has been completely moved to HyperHDR, which uses a LUT table for this purpose (no calculations when converting from YUV to RGB). Using the selected LUT, the YUV to RGB conversion and tone mapping can also be performed in one step without loss. This means that we reduce the client load, as not only is the processor load reduced, but no additional resources, e.g., memory, need to be allocated. You also do not need to use external YUV libraries.
Reduced load on the connection bandwidth. The NV12 frame is around 50% smaller than the RGB frame.
This means that the automatic LUT calibration with YUV coefficient detection can be fully utilized.

# Important!

If the NV12 transport mode is used, HyperHDR requires a complete standard LUT table (150 MB). The small flat_lut_lin_tables.3d table used (50 MB) does not contain the necessary YUV conversion tables and only works for RGB tone mapping.

Another problem was the use of a LUT table for different video types. With SDR video material, for example, the colors of the LEDs matched, but with HDR the colors were distorted and very weak with Dolby Vision.
Thanks to a new calibration method implemented in HyperHDR, it is now possible to generate LUT tables with different color information depending on the video material being played.

The following lines of code were added under other changes, by @s1mptom and @sundermann to the definition of the LUT table in the hyperion-webos backend (PicCap).

Source code:
```
   * #define LUT_TABLE_FILENAME_HDR "lut_lin_tables_hdr.3d"
   * #define LUT_TABLE_FILENAME_DV "lut_lin_tables_dv.3d"
   * #define LUT_TABLE_FILENAME_SDR "lut_lin_tables.3d"
```

In our case, this means that the new LUTs (lut_lin_tables.3d, lut_lin_tables_dv.3d and lut_lin_tables_hdr.3d) replace the standard LUTs.

Thanks to the implementation of a new ZSTD compression process by @awawa-de, compressed LUTs are now permitted and available without any loss of quality. They are automatically detected and applied in real time.

	* lut_lin_tables_hdr.3d.zst
	* lut_lin_tables_dv.3d.zst
	* lut_lin_tables.3d.zst
I recompiled the official HyperHDR-WebOS-Loader and added the following new functions and features:

* Add Settings Panel with Lut install selector by @neogeo
* Fix HyperHDR daemon when starting calibration process by @satgit62
* Fix HyperHDR daemon at screensaver start (The HyperHDR daemon is no longer stopped when the internal screensaver or the custom Aerial Screensaver for webOS is switched on. (Ambilight works) by @satgit62

# The following new features have been added to HyperHDR 22.0.0beta0 by @awawa-dev:

# UPDATE from 10/09/2025:

* Fix Philips Hue Handshake timeout (OpenSSL1.1)
* Infinite Color Engine: add new Exponential smoothing
* Infinite Color Engine: add smoothing test procedure
* Infinite Color Engine: perform smoothing after post-processing
* The Hue lamp HyperHDR driver has been significantly rewritten. Protection against inaccuracies when calculating very dark colors has been added to the algorithms.
* "Candy gamma" is now permanently enabled, as it should be.
* A bug in the wizard has been potentially fixed:Hue setup doesn't let me click "Save"
* Processing (the entire color calibration pipeline) was previously disabled for colors and effects; it is now disabled only for colors.
* Processing: Added the "Scale color output" option (multiplies colors by a user-specified factor).
* Processing: Added the "Limit power output" option. This works a bit like scaling, but dynamically when a given frame exceeds a specified limit. Please do not confuse this with a power limiter for your power supply, where you can set a specific value in watts. We use an abstract 0-1 scale, not watts, and it only regulates the maximum brightness of scenes.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
* Infinite Color Engine
* New Smoothing Algorithms
* Full support for ftdi and libftdi
* Support for Home Assistant lights
* Experimental direct support for zigbee2mqtt
* Support for ZSTD compression for all LUT files
* Added support for NV12 image format for flat buffers by @awawa-dev
* ai_calibration by @awawa-dev
* Denoising algorithm that significantly improves calibration results with YUV420 format, especially YUV420 LIMITED by @awawa-dev
* Added Quarter of frame for NV12 for Flatbuffers Server reduces CPU and memory load by @awawa-dev
* Grabber benchmark support for flat buffers by @awawa-dev
* Added support for SDR (BT2020 in SRGB) calibration by @awawa-dev
* More precise saving/loading of color images by @awawa-dev
* Corrected calibration for 1280×720 recording settings by @awawa-dev

# UPDATE from 16/02/2025: 
Version 0.5.1 of PicCap with the new NV12 backends  has been released. YUV422 to NV12 conversion has been added. 

# To avoid errors, please do not use any other version!

https://github.com/TBSniller/piccap/releases/download/0.5.1/org.webosbrew.piccap_0.5.1_all.ipk

After successful installation and Services Elevation, you need to reboot, open PicCap, configure to your liking, enable NV12, enable autostart, save and reboot.


![PicCap GUI](https://github.com/user-attachments/assets/538d6912-9c05-4574-80b2-189de86a2989)


# Attention! 
When using the NV12 option with multiple LUTs, please do not activate the "Disable HyperHDR SDR/HDR switch" option in the advanced settings of PicCap under any circumstances.

<img width="1451" height="817" alt="advances settings" src="https://github.com/user-attachments/assets/c396aed0-5bb0-4c01-9165-46cb50c540a9" />


# UPDATE from 05/09/2025:

Since HyperHDR developer @awawa-dev has switched to QT 6.8.3, I have decided to also switch the HyperHDR loader for webOS to QT 6.8.3 with the help of @awawa-dev. This will prevent future compatibility issues. It also ensures that this branch is 100 percent compatible with the new features already integrated in this version as well as with future features, and that no components of the app infringe on copyright. 

# 2. HyperHDR installation instructions:

How to install .ipk files without Homebrew Channel can be found in my other guide. See: https://github.com/satgit62/How-to-Install-and-set-up-Ambilight-on-LG-webOS#manual-installation-of-apps-without-homebrew-channel.

# To avoid errors, Please use only this version for these instructions:

https://github.com/satgit62/satgit62.github.io/releases/download/v0.2.0-alpha/org.webosbrew.hyperhdr.loader_0.3.1_all.ipk

To avoid errors, please uninstall HyperHDR webos-loader if you have installed a previous version. Your settings will be preserved.

* a. Start HyperHDR app and stop the services/daemon.
* b. Start webOS Dev Manager on the PC and uninstall HyperHDR.
* c. Install the new webos HyperHDR loader.
* d. Start the HyperHDR application and go to the settings in the top right corner. (gear wheel) See image.
  
<img width="1445" height="817" alt="HyperHDR Loader" src="https://github.com/user-attachments/assets/dcccaabb-1c42-40a5-95a8-86bf09318b77" />

* e. The next step is to select the LUT version to install. Compressed and uncompressed LUT versions are available for NV12. By default, only one LUT is installed and NV12 must be disabled in PicCap.

![2](https://github.com/user-attachments/assets/2640dc01-26d6-4709-a0d9-7cf1c6e9d667)

After you have selected/approved the LUT, wait until Success! appears in the bottom left corner of Result.

![3](https://github.com/user-attachments/assets/57d5c7fa-19d0-4a10-a770-7e7be84ea344)

![4](https://github.com/user-attachments/assets/3e6ad088-6e8f-4344-b8f6-c1847e99964a)

* f. Click on the left arrow in the top left-hand corner of the menu to go to the main menu.
* g. Use the slider to activate the Automatic start option.
* h. Start the daemon by clicking on Start. Wait until the start is displayed.
* i. Reboot. Make sure that Quick Start+ in the menu of your LG is switched off.

The uncompressed LUTs are installed to `/media/developer/apps/usr/palm/services/org.webosbrew.hyperhdr.loader.service/hyperhdr/` and the corresponding symlinks are set to `/home/root/.hyperhdr/`.
The compressed LUTs will be installed in `/home/root/.hyperhdr/`.

# Note

If NV12 mode has been successfully activated, this is displayed in the HyperHDR log:

`[FLATBUFSERVER] (FlatBuffersServer.cpp:xxx) Received first NV12 frame and instead of “(LutLoader.cpp:x) Index 0 for HDR RGB”, “(LutLoader.cpp:x) Index 1 for HDR YUV”` 

When switching from an SDR video to an HDR or DV video, the LUT requested by the backend is searched for, loaded into HyperHDR and displayed in the log:

```
[FLATBUFSERVER] Setting user LUT filename to: 'lut_lin_tables_dv.3d'
[FLATBUFSERVER] (LutLoader.cpp:82) LUT file found: /home/root/.hyperhdr/lut_lin_tables_dv.3d
```

# Remark:
SDR to HDR tone mapping must always be activated for NV12. Also for SDR content. This is how it should look for SDR, but also for HDR or DV content.

![1](https://github.com/user-attachments/assets/4fbf8259-8770-48a6-aa1c-0dd949832947)

# Attention!
Some devices absolutely need 360x180. In this case, do not activate ‘Quarter of image for NV12’.

The default PicCap resolution of 360x180 does not match the actual proportions of 4K video content, so you will need to manually enter 384x216 if you have a newer webOS. 
Otherwise, you will need to select or manually enter 4K resolutions that can be divided by 10.
If you are unsure, try everything from this table.

The most common resolutions for PicCap are:
```
384x216
360x180
256x144
192x108
160x90
128x72
```

# Note: It has been reported that the LG option “Instant Game Response” prevents the correct LUT from being loaded.

# LUT 

If you decide to calibrate the LUTs yourself, make sure you set the resolution in PicCap/Hyperion Webos to at least 1280x720, better 1920x1080, save and restart.

As the calibration is very CPU intensive, I recommend to calibrate on an external HyperHDR, e.g. on Windows or Raspberry Pi.
The external IP address of the HyperHDR has to be temporarily entered in PicCap.

The new function in HyperHDR “Quarter of frame for NV12” under Network, Flatbuffers server must also be switched off for this process. (otherwise you will get the minimum requirement for calibration of 1280 × 720 in HyperHDR)

After a successful calibration, you can reduce the resolution in PicCap and reactivate the option ‘Quarter of image for NV12’ in HyperHDR only if the CPU load is too high.

Comment: Enabling 'Quarter of image for NV12' will reduce the CPU load, but the image resolution will be reduced accordingly. However, this is enough to control the LEDs correctly.

# Note on calibration videos:
SDR refers to content without HDR/HLG or Dolby Vision. For this, you also have to use a calibration video such as calibration_HDR_yuv420_limited_range.mp4, calibration_SDR_yuv420_limited_range.mp4, for_testing_after_calibration_HDR_yuv420_limited_range.mp4

For DV calibration, use the `test_DV_yuv420_low_quality.mp4` file. See above in Main 

So take the appropriate videos for the desired LUT calibration.

Using different players and test files (calibration videos) may slightly affect the calibration result and color reproduction, so it is better to test different combinations if something does not fit.

LUT_calibration_1:

![LUT_calibration](https://github.com/user-attachments/assets/4fa881f3-ae17-4281-bc03-19e2c14f9772)

LUT_calibration_2:

![LUT_calibration1](https://github.com/user-attachments/assets/97cddfb5-4627-4b61-a29a-fabfa96e6b06)


# Important!
At the end of calibration, the LUT is automatically saved in `/home/root/.hyperhdr/` under the name lut_lin_tables.3d.

If the calibration was not performed for SDR, the LUT must also be renamed with the correct name lut_lin_tables_hdr.3d or lut_lin_tables_dv.3d and moved to the correct directory.

See also the calibration guide from @awawa-dev:
https://github.com/awawa-dev/HyperHDR/wiki/lut-calibration

If you would like to compress your self-calibrated LUTs, please read the following instructions:: https://github.com/satgit62/How-to-compress-the-LUT-table-and-use-it-in-HyperHDR


# HyperHDR Image Processing

Finally, you can make fine adjustments such as RGB-gamma, RGB-Temperature or saturation under Image Processing using calibrated colour templates. Colour templates are located at the top of Main.

You must make the fine adjustments under Remote Control Live Calibration and if the results are good, then you must enter and save the values in HyperHDR under Image Processing.

# Live calibration

<img width="1804" height="852" alt="Live calibration" src="https://github.com/user-attachments/assets/14258954-3ea5-475e-8c88-3f690dcb056c" />


# Image Processing

If the colors are not intense enough, increase the saturation. Increase or decrease the gamma values gradually until the desired effect is achieved.
Adjust the temperature according to your LED type.

<img width="1912" height="1059" alt="Infinite Color Engine1" src="https://github.com/user-attachments/assets/8c9bfc5f-6eea-46da-87fe-ab7d9e008b47" />


# Source:

Download org.webosbrew.hyperhdr.loader: https://github.com/satgit62/satgit62.github.io/releases/download/v0.2.0-alpha/org.webosbrew.hyperhdr.loader_0.3.1_all.ipk

Download three test LUT tables for SDR, HDR+ and DV are available for download at the following link: https://drive.google.com/file/d/1gMcooXS14cIQcl5wXywo37i1imaBJGjW/view?usp=sharing

Download Video calibration test files: https://github.com/awawa-dev/awawa-dev.github.io/tree/master/calibration

https://github.com/webosbrew/dev-manager-desktop

# HyperHDR version:

<img width="1623" height="770" alt="OpenSSL 1 1 1q" src="https://github.com/user-attachments/assets/ee0d7d45-7287-45b2-828a-1cbdcd21352b" />

# NV12 Mode Log:

![NV12 Mode Log](https://github.com/user-attachments/assets/76b22449-62e2-4f46-8f95-3733e1540e67)

# Quarter of frame for NV12:

![Quarter of frame for NV12](https://github.com/user-attachments/assets/ae2b4746-7d93-40ac-b8b9-cdaa16475116)

Testdateien_calibration

![Testdateien_calibration](https://github.com/user-attachments/assets/4b8d48fd-3878-4fbb-98d1-bf67d3177c63)

Video benchmark:

![Video benchmark](https://github.com/user-attachments/assets/88d58dfd-fa7e-411a-bdc7-c65090aeacc6)

NV12 coefficients:
See https://gist.github.com/Jim-Bar/3cbba684a71d1a9d468a6711a6eddbeb#about-yuv-formats

![NV12 coefficients](https://github.com/user-attachments/assets/bf8ee0b6-81d3-445f-84d2-3b8725b20581)

