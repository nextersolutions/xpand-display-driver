# xPand Display Driver

The virtual display driver used by [xPand](https://nextersolutions.com/products/xPand) for Windows. It is an IddCx indirect display driver (user-mode, UMDF 2): the xPand app asks it for a monitor sized like the TV, Windows extends the desktop onto it, and xPand captures and streams that monitor to a Google TV.

Forked from [SudoVDA](https://github.com/SudoMaker/SudoVDA), which is based on the Microsoft Indirect Display Driver sample. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## What changed from SudoVDA

- Own identity: hardware ID `Root\XPandDisplay`, device interface `{188abfe0-e836-4ec0-932c-4158485a937c}`, provider "Nextersolutions", EDID manufacturer `XPD`.
- Device access limited to SYSTEM, Administrators and interactive users (SudoVDA allowed Everyone).
- `VIRTUAL_DISPLAY_ADD_PARAMS` carries the TV's physical size in millimetres, written into the EDID so Windows chooses a sensible scale.
- A plain-text log at `%ProgramData%\xPand\driver.log` (written only if the folder grants LOCAL SERVICE write access; the xPand installer sets that up).
- IOCTL protocol version 1.0.0.

The IOCTL contract is [`Common/Include/xpand-display-ioctl.h`](Common/Include/xpand-display-ioctl.h). The xPand app mirrors it in C#; bump the protocol version on any change.

## Build

Visual Studio 2022 with the Desktop C++ workload, Spectre-mitigated libraries and the Windows Driver Kit (WDK 10.0.26100).

```
MSBuild Driver\XPandDisplay.sln /p:Configuration=Release /p:Platform=x64 /p:SignMode=Off
```

Output: `Driver\x64\Release\XPandDisplay\` (`XPandDisplay.dll`, `XPandDisplay.inf`, `xpanddisplay.cat`). Sign the catalog before installing; the xPand repository's `XPand.DriverSetup` installs, repairs and removes the driver.

## Configuration

Optional values under `HKLM\SOFTWARE\Nextersolutions\xPandDisplay`, read when the driver loads:

- `gpuName` (string): friendly name of the GPU to render on. Unset picks the adapter with the most video memory.
- `maxMonitors` (DWORD): maximum virtual monitors. Default 10.
- `watchdog` (DWORD): seconds without any IOCTL before all monitors are removed. Default 3; 0 disables it.
- `sdrBits` (DWORD): 8 (default) or 10.
- `hdrBits` (DWORD): 10 (default) or 12.

Reload the driver (or reboot) after changing them.

## License

MIT, see [LICENSE](LICENSE). Portions derived from the Microsoft sample are under the Microsoft Public License; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
