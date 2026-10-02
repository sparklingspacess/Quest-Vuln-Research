# Quest-Vuln-Research
Research I'm doing for vulnerabilities in Oculus/Meta Quest headsets.

> [!WARNING]
> Please don't take everything in this repo for granted, who knows when they will be patched


Research is being done solely on a Quest 3S (panther)

## Table of Contents

- [Browser](#questBrowser)
- [Working Root](#WorkingRootMethod02/10/26)


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

## WorkingRootMethod02/10/26:
*come back later*
