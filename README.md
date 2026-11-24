# LineageOS 22.2 for Samsung Galaxy A6 (2018) (`a6lte`)

Local manifest for building unofficial LineageOS 22.2 (Android 15) for the
Samsung Galaxy A6 (2018) (`a6lte`, SM-A600F/FN).

It adds the device, kernel, vendor and BSP projects on top of the LineageOS
22.2 manifest. The resulting builds are published in
[Exynos7870-labs/lineage_release](https://github.com/Exynos7870-labs/lineage_release/releases).

## Requirements

- x86_64 Linux host (tested on Linux Mint 22 / Ubuntu 24.04)
- 4 CPU cores minimum, 8 or more recommended
- 16 GB RAM minimum, 32 GB recommended
- ~300 GB free disk space on an SSD (source tree + `out/`)
- A fast internet connection for the initial sync (tens of GB)

No separate toolchain or NDK has to be installed. The source tree ships the
toolchains that are used: a prebuilt Clang for the platform and the prebuilt
GCC 4.9 (`prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9`) for the
Linux 3.18 kernel. Only the host packages below and the `repo` tool are
needed.

### Install host packages (Ubuntu / Linux Mint / Debian)

```bash
sudo apt update
sudo apt install -y \
    bc bison build-essential ccache curl erofs-utils flex g++-multilib \
    gcc-multilib git git-lfs gnupg gperf imagemagick kmod protobuf-compiler \
    python3-protobuf lib32readline-dev lib32z1-dev libdw-dev libelf-dev \
    libgnutls28-dev lz4 libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop \
    pngcrush rsync schedtool squashfs-tools xsltproc xxd zip zlib1g-dev \
    python3 python3-openssl python3-cryptography
```

(On Ubuntu 26.04 and newer, `libxml2` is called `libxml2-16`.)

Install the `repo` launcher and configure git:

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
# ~/bin is normally already in PATH; if not, add it and re-login.

git lfs install
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Optional: enable ccache

```bash
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
```

Add the two `export` lines to `~/.bashrc` to keep them across sessions.

## 1. Initialize the source tree

```bash
mkdir -p ~/android/lineage-22.2
cd ~/android/lineage-22.2

repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs

git clone -b lineage-22.2 \
    https://github.com/samsungexynos7870/android_manifest_samsung_a6lte.git \
    .repo/local_manifests

repo sync -c -j$(nproc) --force-sync --no-clone-bundle --no-tags
```

`repo sync` can be re-run at any time to pick up new changes to the device
trees.

## 2. Generate signing keys (recommended)

The published zips are signed with the Exynos7870-labs private keys, which are
not in any public repository. Without keys the build still succeeds, but it is
signed with the public AOSP test keys: it cannot be installed as an OTA over an
Exynos7870-labs release, and switching between the two requires a clean flash.
Generating your own keys makes your own builds update-compatible with each
other.

```bash
git clone https://github.com/ItsVixano/android_vendor_lineage-priv_keys.git \
    vendor/lineage-priv/keys
cd vendor/lineage-priv/keys
git checkout 4960960cddeae41a36d1d74ed38dafcb6a15c06e  # Android 15 / LineageOS 22.2

python3 -c 'import OpenSSL, cryptography'  # must not fail
```

Edit `gen_keys_py/config.py` and set `SUBJECTS_PARAMS` to your own
name/organisation, then generate the keys:

```bash
python3 ./gen_keys.py
```

The private keys are written to `~/.android-certs`; the signing material and
the regenerated `keys.mk` are written to `vendor/lineage-priv/keys`. Back up
both directories offline, encrypted, and never commit them. If you lose them,
you cannot sign updates that install over your existing builds.

The commit above is the last template revision for Android 15. Newer template
revisions target Android 16 (LineageOS 23.x) and may not work with 22.2.

## 3. Build

```bash
cd ~/android/lineage-22.2
source build/envsetup.sh
brunch a6lte
```

`brunch a6lte` is equivalent to `breakfast a6lte && mka bacon` and
selects the `lineage_a6lte-bp1a-userdebug` target. The first build takes
several hours; later builds are much faster.

The signed, flashable zip is written to:

```
out/target/product/a6lte/lineage-22.2-<date>-UNOFFICIAL-a6lte.zip
```

A copy is also placed at `out/target/product/a6lte/lineage_a6lte-ota.zip`.
This is the archive you flash or publish as a release.

- Rebuild after changing sources: `mka bacon`
- Full clean build: `m clobber` followed by `brunch a6lte`

## 4. Install

- The device needs the additional `/misc` partition, otherwise it may not
  boot. Use the repartition script from
  [universal_repartition_script7870](https://github.com/Exynos7870-labs/universal_repartition_script7870/releases/tag/lineage-22.2).
- TWRP versions that mishandle FBE v1 can stop booting after a flash. Use the
  LineageOS recovery from the release or a current TWRP build
  ([TWRP 3.7.1 Exynos7870](https://sourceforge.net/projects/batuhan-s-builds/files/TWRP-3.7.1-Exynos7870-Stable/)).
- Flash the zip in TWRP or LineageOS recovery. A clean flash (format data) is
  required when switching between differently signed builds, for example
  between an Exynos7870-labs release and your own build.

## What this manifest adds

- `device/samsung/a6lte`, `device/samsung/universal7870-common`
- `kernel/samsung/exynos7870` (Linux 3.18.140)
- `vendor/samsung/a6lte`, `vendor/samsung/universal7870-common`
- Samsung SLSI BSP projects (`hardware/samsung_slsi-linaro/*`)
- Legacy compatibility patches (`vendor/samsung/universal7870-common-patches`)

## Credits

- @Astrako (2021)
- @FlominatorGD (2022)
- The LineageOS project

## Contact

Telegram support group: **deprecated** — https://t.me/joinchat/D1Jk_VbieGBXOWZt2y8O7A
