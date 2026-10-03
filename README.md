# Quest-Vuln-Research
Research I'm doing for vulnerabilities in Oculus/Meta Quest headsets.

> [!WARNING]
> Please don't take everything in this repo for granted, who knows when they will be patched


Research is being done solely on a Quest 3S (panther) and Quest 1 (monterey)

## Table of Contents

- [Browser](#questBrowser)
- [Monterey Browser UAF](#montereyBrowserUaf)

  
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>


## questBrowser
The Quest Browser has a lot of rules, most of them.. shouldn't be given to a browser.
# List of rules (hopefully exploitable):

`allow oculus_browser_app shell_exec:file` - Shell execution

`allow oculus_browser_app toolbox_exec:file` - I don't even know.

`allow oculus_browser_app hal_tracking_default:unix_stream_socket` - Access to tracking data (maybe)

`allow oculus_browser_app trackingfidelityservice:unix_stream_socket` - Same as the one above

`allow oculus_browser_app vendor_sysfs_kgsl:file` - Some sort of access to the GPU Driver (kgsl)

`allow oculus_browser_app vendor_sysfs_kgsl_proc:file` - Same as the one above

# List of rules (unlikely exploitable):
`allow oculus_browser_app usb_device:chr_file` - USB Device access

`allow oculus_browser_app tun_device:chr_file` - Network Tunelling

`allow oculus_browser_app persist_cal_file:file` - Access to calibration files/data

`allow oculus_browser_app system_cal_file:file` - Same as the one above

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

## montereyBrowserUaf

This was initially discovered by Seunghyun Lee (@0x10n), Pwn2Own Vancouver 2024
# Basic Summary:
CVE-2024-2886 is a Use-After-Free in the WebCodecs API. Calling `copyTo()` on a *VideoFrame* and immediately calling `close()` creates a race where the async readback task accesses freed memory. On GPU-backed frames (created from a WebGL canvas), this is confirmed to trigger on Quest 1.
<br>
<br>
Using varying WebGL colors per frame, the bytes returned by `copyTo()` after `close()` consistently differ from baseline, confirming the GPU-backed path is being hit.
<br>
# Binary Analysis (libchrome.so)
Path: */data/app/com.oculus.browser-PEEKvmlKNSWbcUjnSJMdLA==/lib/arm64/libchrome.so*

BuildID: *f61a38a377e959b73e678fce921e86db5a9ac3e2*
<br>
# Vulnerable Function: 0x29357f4

This is the `VideoFrame.copyTo()` *implementation*:

```
0x29357f4  stp x29, x30, [sp, #0x110]
0x2935804  mov x19, x1
0x2935808  mov x20, x0 ; x20 = js VideoFrame wrapper
...
0x29358ac  ldr x20, [x20, #8] ; x20 = inner VideoFrameHandle, this is freed by close()
...
0x29358b8  ldr x8, [x20]  ; vtable pointer from freed object
0x29358cc  ldr x8, [x8, #0x20] ; vtable entry [4]
0x29358d4  blr x8 ; this is a call through to a freed vtable
```
<br>

# Exploit Target

<br>

The vtable dispatch at `0x29358d4` calls through `vtable[4]` of the freed *VideoFrameHandle* object. If the freed memory is reclaimed with attacker-controlled data, `blr x8` executes an arbitrary address.

# References
1. [ZDI-25-027](https://www.zerodayinitiative.com/advisories/ZDI-25-027/) - public advisory
2. [Chromium Issue 330575496](https://issues.chromium.org/issues/330575496) - still restricted, but gave a lead anyway..
3. Fix commit: `webcodecs: Disable async VideoFrame readback to mitigate a race`
4. leesh3288 (@0x10n) - original discoverer, Pwn2Own Vancouver 2024
