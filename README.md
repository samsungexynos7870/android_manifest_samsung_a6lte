# LineageOS 22.2

### How to build ###

```bash
# Create dirs
$ mkdir LineageOS22 && cd LineageOS22

# Init repo
$ repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs
(we have legacy patches)

# Clone my local repo
$ git clone https://github.com/samsungexynos7870/android_manifest_samsung_a6lte.git -b lineage-22.2 .repo/local_manifests

# Sync
$ repo sync --force-sync --no-clone-bundle --no-tags -v

# Build
$ . build/envsetup.sh && brunch lineage_a6lte-bp1a-userdebug
```

## Credits
2021 @Astrako 2022 @FlominatorGD

## Contact
Telegram support group: <s>https://t.me/joinchat/D1Jk_VbieGBXOWZt2y8O7A</s> DEPRECATED
