# LineageOS for Sony Xperia XZ1 Compact (lilac)

## How to build LineageOS

### Initial setup

* Make a workspace:

    ```bash
    mkdir -p ~/lineage
    cd ~/lineage
    ```

    Or use a subfolder for a specific version of LineageOS in a root folder, e.g.

    ```bash
    mkdir -p /lineage/19.1
    cd /lineage/19.1
    ```

* Initialize the repo:


    ```bash
    repo init -u https://github.com/LineageOS/android.git -b lineage-19.1 --git-lfs && git lfs install
    ```

* Create local manifests:

    ```bash
    git clone https://github.com/antoniu200/lineageos_lilac.git
    mkdir .repo/local_manifests
    cd .repo/local_manifests
    ln -s ../../lineageos_lilac/manifests/19.1/*.xml .
    cd -
    ```

* Sync the repo:

    ```bash
    repo sync
    ```

### Build procedure

* Copy [dumped firmware blobs](dump-stock.md) into place for the build

    G8441:
    ```bash
    device/sony/lilac/extract-files.sh /path/to/dumped/firmware
    ```
    
    SO-02K:
    ```bash
    device/sony/lilac/extract-files.sh --dcm /path/to/dumped/firmware
    ```

    We currently use the latest Sony stock, which is `47.2.A.11.228`, so the file will be named like `G8441_*_47.2.A.11.228-*`.

* Apply patches

    ```bash
    device/sony/lilac/patches/applyPatches.sh
    ```
    
* (Optional) Integrate BakaSU v4.2.0-rc3:

    More info [here](https://github.com/JackA1ltman/NonGKI_Kernel_Build_2nd/wiki/en-Embedding-KernelSU-and-Patching-SUSFS).

    ```bash
    cd kernel/sony

    git clone https://github.com/JackA1ltman/NonGKI_Kernel_Build_2nd.git NonGKI_Kernel_Build_2nd
    git -C NonGKI_Kernel_Build_2nd checkout 1790218829

    cd msm8998

    cp ../NonGKI_Kernel_Build_2nd/Bin/clean_hook.sh ./
    cp ../NonGKI_Kernel_Build_2nd/Patches/susfs_inline_hook_patches.sh ./
    cp ../NonGKI_Kernel_Build_2nd/Patches/Patch/susfs_patch_to_4.4.patch ./

    bash clean_hook.sh
    git clone https://github.com/Baka-SU/BakaSU.git KernelSU
    git -C KernelSU checkout v4.2.0-rc3
    curl -LSs https://raw.githubusercontent.com/Baka-SU/BakaSU/refs/tags/v4.2.0-rc3/kernel/setup.sh | bash -s v4.2.0-rc3
    bash susfs_inline_hook_patches.sh
    patch -p1 < susfs_patch_to_4.4.patch

    sed -i '188s/&flags/\&flag/' fs/stat.c
    ```

* Setup the environment

    ```bash
    source build/envsetup.sh
    ```

* Choose the device you're building for

    G8441:
    ```bash
    lunch lineage_lilac-userdebug
    ```

    SO-02K:
    ```bash
    lunch lineage_lilac_dcm-userdebug
    ```

* Tell VNDK we're expanding `libcutils` (1st build only):
    ```bash
    development/vndk/tools/header-checker/utils/create_reference_dumps.py -l libcutils -products lineage_lilac


* Build LineageOS

    ```bash
    mka bacon
    ```
When completed, the built files will be in the `out/target/product/lilac` directory.

### Helper scripts

To simplify the build process I use scripts which are in the [build_scripts](build_scripts) folder, so you can simply run [`build.sh`](build_scripts/build.sh).
They have some assumptions specific to my setup (such as absolute paths) in [`setup.sh`](build_scripts/setup.sh) which may need adjustments for you.
The main [build script](build_scripts/buildAndChecksum.sh) has some additional steps and checks to avoid mistakes in the semi-automated build, but does mostly what is outlined above.
