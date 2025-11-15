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
    repo init -u https://github.com/LineageOS/android.git -b lineage-19.1
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

    ```bash
    cd device/sony/lilac
    ./extract-files.sh /path/to/dumped/firmware
    ```

    We currently use the latest Sony stock, which is `47.2.A.11.228`, so the file will be named like `G8441_*_47.2.A.11.228-*`.

* (Semi-)optionally apply patches

    Some of the patches in this repo fix a few bugs or issues in LineageOS while others make the build deviate a lot from the "vanilla build".
    So this is only for advanced users!

    ```bash
    device/sony/lilac/patches/applyPatches.sh
    ```

    To apply only the minimal (required & security fix) patches:

    ```bash
    device/sony/lilac/patches/applyPatches.sh --minimal
    ```

    To apply only the minimal & new Clang patches:

    ```bash
    device/sony/lilac/patches/applyPatches.sh --minclang
    ```

* Setup the environment

    ```bash
    source build/envsetup.sh
    lunch lineage_lilac-userdebug
    ```

* Build LineageOS

    ```bash
    mka bacon
    ```
When completed, the built files will be in the `out/target/product/lilac` directory.

### Helper scripts

To simplify the build process I use scripts which are in the [build_scripts](build_scripts) folder, so you can simply run [`build.sh`](build_scripts/build.sh).
They have some assumptions specific to my setup (such as absolute paths) in [`setup.sh`](build_scripts/setup.sh) which may need adjustments for you.
The main [build script](build_scripts/buildAndChecksum.sh) has some additional steps and checks to avoid mistakes in the semi-automated build, but does mostly what is outlined above.
