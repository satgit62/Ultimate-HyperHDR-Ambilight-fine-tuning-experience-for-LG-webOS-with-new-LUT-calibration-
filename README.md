# Ultimate-HyperHDR-Ambilight-fine-tuning-experience-for-LG-webOS-with-new-LUT-calibration-

This guide here is based on @awawa-dev New LUT Calibration implementations in HyperHDR “ai_calibration” branch and @s1mptom changes in hyperion-webos fork “dile_vt_422_nv12” branch.

The implementation and utilization of the NV12 image format in HyperHDR now offers numerous enhancements and the capability to achieve precise color matching even in our exotic LG webOS by utilizing the YUV format in lieu of RGB.

The first step was achieved by @awawa-dev by implementing NV12 in HyperHDR.

# Advantages:

The NV12-YUV conversion has been completely moved to HyperHDR, which uses a LUT table for this purpose (no calculations when converting from YUV to RGB). Using the selected LUT, the YUV to RGB conversion and tone mapping can also be performed in one step without loss. This means that we reduce the client load, as not only is the processor load reduced, but no additional resources, e.g., memory, need to be allocated. You also do not need to use external YUV libraries.
Reduced load on the connection bandwidth. The NV12 frame is around 50% smaller than the RGB frame.
This means that the automatic LUT calibration with YUV coefficient detection can be fully utilized.

# Important!

If the NV12 transport mode is used, HyperHDR requires a complete standard LUT table (150 MB). The small flat_lut_lin_tables.3d table used (50 MB) does not contain the necessary YUV conversion tables and only works for RGB tone mapping.

Another problem was the use of a LUT table for different video types. With SDR video material, for example, the colors of the LEDs matched, but with HDR the colors were distorted and very weak with Dolby Vision.
Thanks to a new calibration method implemented in HyperHDR, it is now possible to generate LUT tables with different color information depending on the video material being played.

The second step was taken by user @s1mptom, in consultation with @awawa-dev, who himself was not entirely satisfied with the results of the modified backend. He adapted the hyperion-webos backend for the NV12 format and also added the use of multiple LUT tables. This finally makes it possible to use automatically calibrated LUT tables for the three different scenarios such as SDR, HDR+ and Dolby Vision video material in HyperHDR.

The following lines of code were added under other changes, by @s1mptom to the definition of the LUT table in the hyperion-webos backend (PicCap).

Source code:

  `#define LUT_TABLE_FILENAME_HDR “lut_lin_tables_hdr.3d”
  #define LUT_TABLE_FILENAME_DV “lut_lin_tables_dv.3d”
  #define LUT_TABLE_FILENAME_SDR “lut_lin_tables.3d”`

In our case, this means that we replace the new LUTs (lut_lin_tables.3d, lut_lin_tables_dv.3d and lut_lin_tables_hdr.3d) with the standard LUT.

For this, @awawa-dev has also added a new LUT calibration option for our WebOS Flatbuffers, which allows us to play different test videos directly on our devices to generate the different LUTs.
YUV coefficients are switched during LUT calibration thanks to the native NV12 format. This was not possible for the pure RGB flat buffer stream.
Even the first LUT calibration for SDR, using the reference test video for SDR, “test_SDR_yuv420_low_quality”, produced a remarkable color correction.

# I have compiled the “hyperhdr-webos-loader” with the following new feature from @awawa-dev and built the installation package based on these changes and the purpose of the calibration:

* Added support for NV12 image format for flat buffers
* ai_calibration:
* Denoising algorithm that significantly improves calibration results with YUV420 format, especially YUV420 LIMITED
* Added Quarter of frame for NV12 for Flatbuffers Server reduces CPU and memory load
* Grabber benchmark support for flat buffers
* Added SDR support for libdile
* Added support for SDR (BT2020 in SRGB) calibration
* More precise saving/loading of color images
* Corrected calibration for 1280×720 recording settings

# 1.replace backends in hyperion-webos (PicCap):

a. Connect to your TV via SSH/Telnet and execute the following command `pgrep -fl piccap | while read -r pid _; do kill “$pid”; done` to terminate piccap with Hyperion process.
b. Unzip the webos_backends.zip.
c. Then you have to replace all new backend files in `/media/developer/apps/usr/palm/services/org.webosbrew.piccap.service/` the old backend files.
d. Then you must execute `luna-send -n 1 -f luna://org.webosbrew.piccap.service/setSettings “{\”use_direct_nv12\“: true}” `in Terminal/SSH to activate NV12 mode.

Note: If NV12 mode has been successfully activated, this is displayed in the log: [FLATBUFSERVER] (FlatBuffersServer.cpp:338) Received first NV12 frame.

# 2. HyperHDR installation instructions:

a. Start HyperHDR app and stop the services/daemon.
b. Start webOS Dev Manager on the PC and uninstall HyperHDR.
c. Install HyperHDR version 21.0.0.0alpha0 Build ai_calibration branch from attachment (org.webosbrew.hyperhdr.loader_0.1.9_all.ipk).
d. Start HyperHDR app and start the services/daemon.
e. Reboot. Make sure that Quick Start+ in the menu of your LG is switched off.
f. Using FileZilla or webOS Dev Manager, save the 50 MB “flat_lut_lin_tables.3d” from `/media/developer/apps/usr/palm/services/org.webosbrew.hyperhdr.loader.service/hyperhdr/` and then delete it from the above-mentioned directory.
g. Using FileZilla or webOS Dev Manager, also delete the “lut_lin_tables.3d” from `/home/root/.hyperhdr/` directory of your TV.
h. Download the LUT.zip, unzip it and copy the contents of the LUT folders lut_lin_tables_hdr.3d and lut_lin_tables_dv.3d to `/home/root/.hyperhdr/` using FileZilla or webOS Dev Manager on the TV and copy lut_lin_tables.3d to `/media/developer/apps/usr/palm/services/org.webosbrew.hyperhdr.loader.service/hyperhdr/`.
i. In HyperHDR under Network Settings, Flatbuffer Server, activate the HDR to SDR sound mapping Quarter of frame for NV12.

When switching from an SDR video to an HDR or DV video, the HDR (global) mode is automatically switched on and the LUT requested by the backend is searched for, loaded in HyperHDR and displayed in the log:

[FLATBUFSERVER] Setting user LUT filename to: 'lut_lin_tables_dv.3d'
[FLATBUFSERVER] (LutLoader.cpp:82) LUT file found: `/home/root/.hyperhdr/lut_lin_tables_dv.3d`

When returning to an SDR video, the HDR (global) mode is automatically switched off and the SDR LUT is requested.

If you decide to calibrate LUTs yourself, make sure that you set the resolution in PicCap/hyperion-webos to Manuel and set 1280 × 720, save and restart.
The new function in HyperHDR “Quarter of frame for NV12” under Network, Flatbuffers server must also be switched off for this process. (otherwise you will get the minimum requirement for calibration of 1280 × 720 in HyperHDR)
After a successful calibration process, you can reduce the resolution in PicCap again and reactivate the “Quarter of frame for NV12” in HyperHDR.

# Note on calibration videos:
SDR refers to content without HDR/HLG or Dolby Vision. For this, you also have to use a calibration video such as test_SDR_yuv420_low_quality or test_SDR_yuv422_low_quality. As YUV422 is not so widespread, I have used test_SDR_yuv420_low_quality.

The same applies to HDR or DV
So take the appropriate videos for the desired LUT calibration.

The HDR to SDR sound mapping automatically controls the HDR (Global) switch, under remote control on or off, when SDR or HDR/DV is played.
The function should be switched off during calibration. Then you have to switch it on again.

# Note from @awawa-dev:
Using different players and test files (calibration videos) may slightly affect the calibration result and color reproduction, so it is better to test different combinations if something does not fit.

# Source:
# Download hyperion-webos von @s1mptom:https://github.com/s1mptom/hyperion-webos/actions/runs/10928329050/artifacts/1949551235
# Download org.webosbrew.hyperhdr.loader_0.1.9_all.ipk:https://github.com/satgit62/hyperhdr-webos-loader/releases/download/latest/org.webosbrew.hyperhdr.loader_0.1.9_all.ipk
# Download three test LUT tables for SDR, HDR+ and DV are available for download at the following link:https://drive.google.com/file/d/1eUXQ8pkaipQVP3nn4IWlnMKyj70w9YvE/view?usp=sharing
# https://gist.github.com/Jim-Bar/3cbba684a71d1a9d468a6711a6eddbeb#about-yuv-formats
# https://github.com/awawa-dev/HyperHDR/pull/920
# https://github.com/awawa-dev/HyperHDR/pull/896
# https://github.com/satgit62/hyperhdr-webos-loader
# https://github.com/s1mptom/hyperion-webos/tree/main
# Video calibration test files:https://github.com/awawa-dev/awawa-dev.github.io/tree/master/calibration




  
