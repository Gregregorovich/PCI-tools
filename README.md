# PCI-tools
Repository of scripts for easily readable information about PCI devices mainly for usage in (a) KVM(s). Includes different ways of listing IOMMU groups, listing kernel drivers, PCIe lanes and speeds for specific PCIe devices, and which USB devices belong to which IOMMU groups


`ls-iommu` lists all PCI devices, with a single line for each device, ordered by their IOMMU group. Source: Level1Techs forum.
```
[...]
IOMMU Group 19 04:06.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43f5] (rev 01)
IOMMU Group 1 00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:14db]
IOMMU Group 20 04:07.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43f5] (rev 01)
IOMMU Group 21 04:08.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43f5] (rev 01)
IOMMU Group 21 0a:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43f4] (rev 01)
[...]
```

`ls-iommu-reset` lists all PCI devices and whether they support reset. Other than highlighting it through grep and making the function multi-line, this command comes from `SpaceInvader One` at Dropbox https://www.dropbox.com/s/o5p7l7lu6qcpezt/commands%20for%20usb%20passthrough.txt.zip?dl=0&file_subpath=%2Fcommands+for+usb+passthrough.txt , which I found through Fenguooerbian's blog https://fenguoerbian.github.io/post/device-passthrough-in-kvm/
```
[...]
IOMMU group 0
        00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:14da]
IOMMU group 19
[RESET] 04:06.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43f5] (rev 01)
IOMMU group 9
[...]
```


`ls-iommu-grouped` lists all PCI devices, with an extra line separating each group. I added the group number indented at the beginning of each PCI device line as otherwise it was useless in grep. I'm not sure where I sourced the original script from, but I suspect it was somewhere on the Level1Techs forum. E.G.:
```
[...]
IOMMU Group 10:
  10:   00:08.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:14dd]
IOMMU Group 11:
  11:   00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 71)
  11:   00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 12:
  12:   00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Device [1022:14e0]
[...]
```

`ls-usb-iommu` lists all USB controllers, buses and devices, grouped by IOMMU, which is especially useful for finding out which USB controllers / hubs can be passed through (though you may have to plug in a unique device to each port and see where it pops up - I annotated my motherboard manual after printing it out). Note that different buses can be in the same IOMMU group:
```
[...]
Bus 3 --> 0000:13:00.0 (IOMMU group 21)
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 002: ID 1462:7d67 Micro Star International [unknown]

Bus 4 --> 0000:13:00.0 (IOMMU group 21)
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

Bus 5 --> 0000:15:00.0 (IOMMU group 22)
Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 005 Device 002: ID 194f:0424 PreSonus Audio Electronics, Inc. [unknown]
[...]

```

`lspci-devices` lists only/all devices by ID (listed as arguments), showing PCI ID, Device ID, kernel driver in use, and whether the device supports reset. E.G.:
```
lspci-devices 1022:15b6 1022:15b7
1a:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:15b6]
        Kernel driver in use: vfio-pci
        Supports RESET: YES
1a:00.4 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:15b7]
        Kernel driver in use: vfio-pci
        Supports RESET: YES
```

`lspci-lanes` is a more verbose variant, that lists PCI ID, Device ID, kernel driver in use, the number of PCIe lanes in use, PCIe speed (and generation can be interpreted from there), IOMMU group, and whether the device supports reset. It also uses grep to highlight the device ID, the PCIe speeds and number of lanes. E.G.: (You'll notice that while my 5600XT has a x16 link to a separate PCI device - the "Upstream Port of PCI Express Switch" - that only has 4 lanes connecting it to the system)
```
lspci-lanes 10de:1f02 1002:1478 1002:731f 1002:164e
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU106 [GeForce RTX 2070] [10de:1f02] (rev a1) (prog-if 00 [VGA controller])
                LnkCap: Port #0, Speed 8GT/s, Width x16, ASPM L0s L1, Exit Latency L0s <1us, L1 <4us
                LnkSta: Speed 8GT/s, Width x16
                LnkCap2: Supported Link Speeds: 2.5-8GT/s, Crosslink- Retimer- 2Retimers- DRS-
        Kernel driver in use: vfio-pci
        Kernel modules: nouveau
        IOMMU Group: 13
        Supports RESET: YES
17:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 XL Upstream Port of PCI Express Switch [1002:1478] (rev ca) (prog-if 00 [Normal decode])
                LnkCap: Port #2, Speed 16GT/s, Width x16, ASPM L1, Exit Latency L1 <64us
                LnkSta: Speed 16GT/s, Width x4 (downgraded)
                LnkCap2: Supported Link Speeds: 2.5-16GT/s, Crosslink- Retimer+ 2Retimers+ DRS-
        Kernel driver in use: pcieport
        IOMMU Group: 24
        Supports RESET: YES
19:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] [1002:731f] (rev ca) (prog-if 00 [VGA controller])
                LnkCap: Port #0, Speed 16GT/s, Width x16, ASPM L0s L1, Exit Latency L0s <64ns, L1 <1us
                LnkSta: Speed 16GT/s, Width x16
                LnkCap2: Supported Link Speeds: 2.5-16GT/s, Crosslink- Retimer+ 2Retimers+ DRS-
        Kernel driver in use: amdgpu
        Kernel modules: amdgpu
        IOMMU Group: 26
        Supports RESET: YES
1a:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Raphael [1002:164e] (rev c2) (prog-if 00 [VGA controller])
                LnkCap: Port #0, Speed 16GT/s, Width x16, ASPM L0s L1, Exit Latency L0s <64ns, L1 <1us
                LnkSta: Speed 16GT/s, Width x16
                LnkCap2: Supported Link Speeds: 2.5-16GT/s, Crosslink- Retimer+ 2Retimers+ DRS-
        Kernel driver in use: amdgpu
        Kernel modules: amdgpu
        IOMMU Group: 28
        Supports RESET: YES
```
