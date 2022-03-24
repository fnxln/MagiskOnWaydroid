# Gapps + Magisk on Waydroid 
[BlissOS/LineageOS 18.1 Dev Image (Android 11)](https://sourceforge.net/projects/blissos-dev/files/waydroid/lineage/lineage-18.1/)

## Bugs
1. **Zygisk not yet working (No ETA)**

    In [LSPosed's MagiskonWSA](https://github.com/LSPosed/MagiskonWSA), there is a patched kernel with su binaries so that zygote process can be patched with zygisk capable zygote. This means su is available before even initrc started. However Waydroid uses lxc containers utilizing linux host kernel without patched su binaries. 
    
    Currently, this script is using MagiskonWSA method in patching initrc so that it would load magisk su binaries so that by the time the UI is loaded, Magisk root manager is ready to use. There might be a way to load su binaries as kernel module when lxc is starting. Anybody who is well-versed in lxc can contact me/create issue to explain to me how to make it works.
    
    Modules requiring zygote/zygisk like [Riru](https://github.com/RikkaApps/Riru), [LSPosed](https://github.com/LSPosed/LSPosed) (pre-zygisk) or Shamiko (module to hide magisk root utilizing zygisk) wont work. Example of modules working: Busybox NDK, Magisk Hide Prop, Detach (detach app from play store)
    
2. **Restart waydroid container twice after additional setup in Magisk.**
   
   First restart usually have a bug where the ethernet connection would fail to connect to the internet. Second restart should fix them.

    ``` 
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid stop session
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid show-full-ui
    ```
    
3. Magisk Canary not guaranteed to work. 
   
   The latest that works is v24.1 up to 24102.
   This is due to a recent change of how Magisk load the binaries when booting somewhere in commit between build v24102 to v24103 and present in v24.2, 2420x builds.
   I might add several links of magisk version so that new version from 2430x can be tested. However, default would still be v24.1.

## Features
- Forked [MagiskOnWSA](https://github.com/LSPosed/MagiskOnWSA) and modified to install Magisk and Pico OpenGapps in Waydroid 11 system.img.
- Support all OpenGApps variants except for aroma (aroma does not support x86_64, please use super instead)


## Text Guide

1. Go to the **Action** tab in your forked repo
    ![Action Tab](https://docs.github.com/assets/images/help/repository/actions-tab.png)
1. In the left sidebar, click the **Build WSA** workflow.
    ![Workflow](https://docs.github.com/assets/images/actions-select-workflow.png)
1. Above the list of workflow runs, select **Run workflow**
    ![Run Workflow](https://docs.github.com/assets/images/actions-workflow-dispatch.png)
1. Input the download link of Magisk and select the [OpenGApps variant](https://github.com/opengapps/opengapps/wiki#variants) (none is no OpenGApps) you like, select the root solution (none means no root) and click **Run workflow**
    ![Run Workflow](https://docs.github.com/assets/images/actions-manually-run-workflow.png)
1. Wait for the action to complete and download the artifact
    ![Download](https://docs.github.com/assets/images/help/repository/artifact-drop-down-updated.png)
1. Unzip the artifact
    - The size shown in the webpage is uncompressed size and the zip you download will be compressed. So the size of the zip will be much less than the size shown in the webpage.
1. Copy system.img and vendor.img to /usr/share/waydroid-extras/images
    ```shell
    # Create dir if not exists, copy files
    sudo mkdir -p /user/share/waydroid-extras/images
    sudo cp system.img /user/share/waydroid-extras/images/
    sudo cp vendor.img /user/share/waydroid-extras/images/
    # Initialize new images
    sudo waydroid init -f
    # Restart waydroid lxc container
    sudo systemctl start waydroid-container.service
    waydroid session start &
    # Start Waydroid UI
    waydroid show-full-ui
    ```
1. After additional setup in magisk, reboot and restart container twice (refer to Bugs as to why)
    ```
    # First container restart
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid stop session
    # Second container restart
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid show-full-ui
    ```
    
## FAQ

- Why the size of the zip does not match the one shown?

   The zip you downloaded is compressed and Github is showing the uncompressed size.
- How can I update System.img to new version?

    Rerun the Github action, download the new artifact, replace the content of your previous installation.
- How can I update Magisk to new version?

    Do the same as updating System.img
- How to pass safetynet?

    Like all the other emulators, no way.
- Magisk online module list is empty?

    Install manually by 
   
    `adb push module.zip /data/local/tmp`
    
    `adb shell su -c magisk --install-module /data/local/tmp/module.zip`.
- Can I use Magisk 23.0 stable or lower version?

    No. Magisk has bugs preventing itself running on WSA. Magisk 24+ has fixed them. So you must use Magisk 24 or higher version.

- Github Action script is updated, how can I synchronize it?

    1. In your fork repository, click `fetch upstream`
        ![fetch](https://docs.github.com/assets/cb-33284/images/help/repository/fetch-upstream-drop-down.png)
    1. Then and click `fetch and merge`
        ![merge](https://docs.github.com/assets/cb-128489/images/help/repository/fetch-and-merge-button.png)

## Credits
- [Waydroid](https://github.com/waydroid/waydroid)
- [Magisk](https://github.com/topjohnwu/Magisk)
- [The Open GApps Project](https://opengapps.org)
- [WSA-Kernel-SU](https://github.com/LSPosed/WSA-Kernel-SU) and [kernel-assisted-superuser](https://git.zx2c4.com/kernel-assisted-superuser/)
- [WSAGAScript](https://github.com/ADeltaX/WSAGAScript)
- [MagiskonWSA](https://github.com/LSPosed/MagiskonWSA)
- [Project Kokoro](https://github.com/supremegamers/kokoro)
- [MagiskOnEmulator](https://github.com/shakalaca/MagiskOnEmulator)
- [MagiskOnEmu](https://github.com/HuskyDG/MagiskOnEmu)
