# ZyNOS Notes

As no public information is available for many currently available ZyNOS versions, this will document what is known or speculated.

**DISCLAIMER**: No information herein is stated to be true. Use at your own risk. Information contained herein is for educational or research use only. It doesn't make me happy to write this, but that's the world we live in.

## What is ZyNOS?

It is ZyXEL's proprietary OS for network-connected devices. As the [Wikipedia page](https://en.wikipedia.org/wiki/ZyNOS) states: the name "is a contraction of _ZyXEL_ and _Network Operating System_ (NOS)".

## Which devices have ZyNOS-based firmware?

Quite a lot, actually, and not only ZyXEL branded ones. Apparently, ZyNOS code is available for licensed use in e.g. SDKs for ADSL chipsets, as can be see with say Trendchip.

## Have they any versioning?

They indeed do. I have observed the following ZyNOS versions as marked in ZyXEL firmware packages:

* 2.50 -- found in old devices like P-782
* 3.40 -- the most popular in ADSL routers
* 3.60
* 3.70
* 3.80
* 3.90
* 4.00 -- for e.g. ZyXEL GS2200

It is currently unknown to me which features are added or removed with each version.

### ZyXEL firmware versioning schema

It is worthy to take notes on firmware versioning schema adopted by ZyXEL (albeit undocumented). Versions have format like `3.60(TX.0)C0` or `3.40(FN.0)C0` and obviously consists of 3 parts.

* Part 1: ZyNOS version, I assume, or that part of the OS that is device-independent. In the two cases above, it is 3.60 and 3.40.
* Part 2: Device-dependent code version. The letters stay the same across the devices, while the digit changes with versions, resetting to zero with each ZyNOS code update.
* Part 3: Unknown; could be configuration or documentation changes only. At least in one case, firmware binaries compared equal (`370TX0C0` and `370TX0C1`).

Note: Firmware released by ZyXEL has the version identified compiled in _BootExtension_.

Note: The same versioning schema is used for non-ZyNOS based products as well (for example, Linux-based devices). So jugding by version number only it is not possible to tell whether firmware is based on ZyNOS or not.

## On firmware structure

Every ZyNOS-running device posesses a ROM chip containing the firmware image. Typically, this chip is mapped into memory space starting at `BFC00000` (on MIPS-based devices), but that is subject to a specific platform.

The image is divided into sections or _objects_. The mapping of objects to addresses in ROM is achieved via the _memory mapping table_, which itself is stored in ROM. The table stores at least the object's name, starting address, size, and type. Note the table maps both ROM and RAM memory sections, so it is possible to figure out memory configuration of the whole device, not only ROM.

RAM object contents are never contained within update image -- quite obviously.

### Memory mapping and objects

All inspected devices so far contain at least these objects: `MemMapT`, `BootBas`, `BootExt`, `RasCode`, `RomDefa`, `termcap`.

* The `MemMapT` object maps the memory mapping table -- its actual location and size in ROM.
* The `BootBas` object maps the _BootBase_ code -- the initial program loader for the device. It is not actually contained within the firmware update image, but I have seen a few firmware releases from ZyXEL that contain _BootBase_ update in a separate file. Apart from boot code, _BootBase_ contains vendor and model names. _BootBase_ is rather small, typically 16K, but then, it does not need to do much except loading stage 2.
* The `BootExt` object maps the _BootExtension_ code -- stage 2 program loader. It also contains rudimentary debugging facilities allowing to recover the device in case of e.g. problems with configuration. _BootExtension_ is responsible to load actual ZyNOS code.
* The `RasCode` object contains the OS image (named RAS -- acronym?) -- the final stage.
* The `RomDefa` object contains ROMFILE with default configuration settings. 
* The `termcap` object contains what looks like, well, termcap description. I am not sure this is _actually_ used anywhere in code.

Objects with unknown contents:

* `DbgArea`
* `RomDir2`

Depending on the device, the following objects may be present:

* The `HTPCode` object contains _Hardware Test Program_, which can be loaded via _BootExtension_

An example memory mapping looks like this:

```
Object: 'BootExt ' at 80008000, size 00018000 (RAMBOOT, 0)
Object: 'HTPCode ' at 80020000, size 000E0000 (RAMCODE, 0)
Object: 'RasCode ' at 80020000, size 00340000 (RAMCODE, 0)
Object: 'BootBas ' at BFC00000, size 00004000 (ROMIMG, 0)
Object: 'DbgArea ' at BFC04000, size 00002000 (ROMIMG, 1)
Object: 'RomDir2 ' at BFC06000, size 00002000 (ROMDIR, 2)
Object: 'BootExt ' at BFC08030, size 00013FD0 (ROMIMG, 3)
Object: 'MemMapT ' at BFC1C000, size 00000C00 (ROMMAP, 5)
Object: 'HTPCode ' at BFC1CC00, size 00008000 (ROMBIN, 4)
Object: 'termcap ' at BFC24C00, size 00000400 (ROMIMG, 6)
Object: 'RomDefa ' at BFC25000, size 00002000 (ROMIMG, 7)
Object: 'LedDefi ' at BFC27000, size 00000400 (ROMIMG, 8)
Object: 'LogoImg ' at BFC27400, size 00001000 (ROMIMG, 9)
Object: 'LogoImg2' at BFC28400, size 00001000 (ROMIMG, 16)
Object: 'StrImag ' at BFC29400, size 00002000 (ROMIMG, 17)
Object: 'fdata   ' at BFC2B400, size 00002800 (ROMBIN, 18)
Object: 'RasCode ' at BFC2DC00, size 0004A400 (ROMBIN, 19)
```

## Is ZyNOS written from scratch?

Apparently, no. At least some firmware versions for inspected xDSL routers contain ThreadX copyright strings. Namely:

- `ThreadX R3900/Green Hills Version G3.0f.3.0b`
  - AirLive ARM-201, WT2000ARM
  - Billion BiPAC 5100, 5102, 5102S, 5200/SRC/SRD, 5210SRF
  - D-LINK DSL-2640R, DSL-2641R, DSL-2740R
  - Huawei HG520c, HG530
  - TP-LINK TD-8816 V1 through V7
  - ZTE ZXV10 W300
  - ZyXEL ES-2024A firmware versions 3.60 through 3.80
  - ZyXEL ES-3024
  - ZyXEL P-2602HWL-D3A
  - ZyXEL P-2602HW-61
  - ZyXEL P-660HN-T3A, P-660HW-D1, P-660HW-T1
  - ZyXEL P-662H, P-662HW
  - ZyXEL P-791R v1 and v2
- `ThreadX ARM9/Green Hills Version G4.0.4.0`
  - ZyXEL ES-2048 firmware versions 3.70 through 3.80
  - ZyXEL ES-3124 firmware versions 3.50 through 3.80
- `ThreadX MIPS32_M14K/GNU Version G5.5.5.0 SN: 3461-183-0501`
  - TP-LINK TD-8816 V8

Web-based configurator UI is implemented using RomPager, with its versions not correlating with ZyNOS versions. This suggests the configurator is built separately for each product (ES-2024A version 3.70 bundles `RomPager Advanced Version 4.30b3` while P-793H version 3.70 bundles `RomPager/4.51`).

