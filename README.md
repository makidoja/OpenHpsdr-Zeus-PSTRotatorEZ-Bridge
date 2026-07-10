# ZeusPstBridge

**ZeusPstBridge** is a Windows bridge between the Hamlib `rotctld` network interface used by **OpenHPSDR Zeus** and the UDP control interface provided by **PstRotatorAz**.

A single copy of ZeusPstBridge can operate up to **10 independent rotator channels**. Each channel appears to Zeus as a separate `rotctld` TCP server and is linked to its own PstRotatorAz IP address and UDP port pair.

```text
OpenHPSDR Zeus                 ZeusPstBridge                    PstRotatorAz

Rotator 1 ── TCP 4533 ──► Channel 1 ── UDP/XML ──► 192.168.1.101:12000
Rotator 2 ── TCP 4535 ──► Channel 2 ── UDP/XML ──► 192.168.1.102:12000
Rotator 3 ── TCP 4537 ──► Channel 3 ── UDP/XML ──► 192.168.1.103:12100
    ...                         ...
Rotator 10 ─ TCP 4551 ──► Channel 10 ─ UDP/XML ──► another PST instance
```

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Building the program](#building-the-program)
- [Running ZeusPstBridge](#running-zeuspstbridge)
- [Channel settings](#channel-settings)
- [Configuring PstRotatorAz](#configuring-pstrotatoraz)
- [Configuring OpenHPSDR Zeus](#configuring-openhpsdr-zeus)
- [Example installations](#example-installations)
- [Testing without a physical rotator](#testing-without-a-physical-rotator)
- [Supported commands](#supported-commands)
- [Configuration and log files](#configuration-and-log-files)
- [Firewall and network security](#firewall-and-network-security)
- [Troubleshooting](#troubleshooting)
- [Known limitations](#known-limitations)

## Features

- Up to 10 independent rotator channels in one application.
- A separate Hamlib `rotctld` TCP endpoint for every enabled channel.
- A separate PstRotatorAz hostname/IP address, UDP command port and UDP reply port for every channel.
- Supports PstRotatorAz instances on the local PC or elsewhere on the LAN.
- Supports several remote PstRotatorAz computers using the same UDP port pair, provided their IP addresses are different.
- Polls actual azimuth using `<PST>AZ?</PST>`.
- Polls target azimuth using `<PST>TGA?</PST>`.
- Converts absolute azimuth, stop and park commands from Hamlib to PstRotatorAz UDP commands.
- Live display of Zeus connection state, PstRotatorAz state, current azimuth, target azimuth and last reply time.
- Manual Query, Go, Stop, Park and **STOP ALL** controls.
- Multiple simultaneous TCP clients per channel.
- Optional protocol traffic logging for diagnosis and development.
- Automatically saves configuration in the current user's local application-data folder.
- Includes a PstRotatorAz simulator and a simple `rotctld` test client.

## Requirements

### To build from source

- Windows 10 or Windows 11.
- The **.NET 8 SDK**.
- A Windows x64 or Windows ARM64 computer.

Download the .NET 8 SDK from:

```text
https://dotnet.microsoft.com/download/dotnet/8.0
```

Install the **SDK**, not only the Desktop Runtime. The build script creates a self-contained executable, so computers running the finished program do not need .NET installed separately.

### To use the bridge

- OpenHPSDR Zeus with Hamlib/`rotctld` network rotator support.
- One or more configured PstRotatorAz installations.
- IP connectivity between ZeusPstBridge and each PstRotatorAz computer.
- One unused TCP port per Zeus rotator channel.
- One UDP command/reply port pair per PstRotatorAz instance when multiple instances share the same computer.

## Building the program

### Recommended x64 build

1. Extract the source archive to a normal writable folder.
2. Install the .NET 8 SDK.
3. Open the extracted `ZeusPstBridge-v0.1` folder.
4. Double-click:

```text
Build-Windows.bat
```

Alternatively, open PowerShell in the project folder and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\Build-Windows.ps1
```

The finished program will be created at:

```text
publish\win-x64\ZeusPstBridge.exe
```

The published application is a self-contained, single-file Windows executable.

### Windows ARM64 build

Run:

```powershell
powershell -ExecutionPolicy Bypass -File .\Build-Windows.ps1 -Runtime win-arm64
```

The result will be placed in:

```text
publish\win-arm64\ZeusPstBridge.exe
```

### Build from the command line

Developers can also build the solution directly:

```powershell
dotnet restore .\ZeusPstBridge.sln
dotnet build .\ZeusPstBridge.sln --configuration Release
```

To publish an x64 self-contained executable manually:

```powershell
dotnet publish .\ZeusPstBridge\ZeusPstBridge.csproj `
  --configuration Release `
  --runtime win-x64 `
  --self-contained true `
  --output .\publish\win-x64 `
  /p:PublishSingleFile=true `
  /p:IncludeNativeLibrariesForSelfExtract=true
```

## Running ZeusPstBridge

Run:

```text
ZeusPstBridge.exe
```

On first launch, the program creates a default configuration containing 10 rows. Only **Rotator 1** is enabled initially.

The default first channel is:

```text
Name:             Rotator 1
Zeus listen IP:  127.0.0.1
Zeus TCP port:   4533
PST IP / host:   127.0.0.1
PST command port: 12000
PST reply port:   12001
Poll interval:    500 ms
Azimuth range:    0 to 360 degrees
```

Configure the required rows, tick **On** for each channel that should run, and press **Start Bridge**.

Starting the bridge automatically saves the settings shown in the grid. To change channel settings later, stop the bridge, edit the rows, then press **Save & Apply** or **Start Bridge**.

### Main controls

| Control | Function |
|---|---|
| **Start Bridge** | Saves the displayed configuration and starts all enabled channels. |
| **Stop Bridge** | Stops the TCP listeners and UDP polling. It does not deliberately move the rotators. |
| **Save & Apply** | Saves changes. If the bridge is running, enabled channels are restarted with the new settings. |
| **STOP ALL** | Sends `<PST><STOP>1</STOP></PST>` to every enabled channel. |
| **Query Selected** | Immediately requests the current and target azimuth for the selected row. |
| **Set AZ / Go** | Sends the entered absolute azimuth to the selected channel. |
| **Stop Selected** | Sends a stop command to the selected channel. |
| **Park Selected** | Sends the PstRotatorAz park command to the selected channel. |
| **Start bridge when program opens** | Automatically starts all enabled channels when the program launches. |
| **Protocol traffic log** | Records Hamlib commands, Hamlib responses and PstRotatorAz UDP traffic. |
| **Open Config Folder** | Opens the folder containing `config.json` and the Logs folder. |

## Channel settings

Each row represents one completely independent rotator connection.

| Column | Meaning |
|---|---|
| **On** | Enables the channel when the bridge starts. |
| **Name** | Friendly name shown in the bridge log, such as `2 m`, `70 cm` or `Dish`. |
| **Zeus listen IP** | Local IPv4 address on which the bridge accepts Zeus/rotctld TCP connections. |
| **TCP** | Unique TCP port used by Zeus for this rotator. |
| **PST IP / host** | IP address or DNS hostname of the computer running the matching PstRotatorAz instance. |
| **PST port** | PstRotatorAz UDP command port. |
| **Reply** | Local UDP port on which PstRotatorAz replies are expected, normally command port + 1. |
| **Poll ms** | Time between current-position requests. Valid range: 200 to 10,000 ms. |
| **Min AZ** | Lowest azimuth the bridge will accept from Zeus or the manual Go control. |
| **Max AZ** | Highest azimuth the bridge will accept. |
| **Zeus** | Runtime status: Stopped, Listening, or the number of connected clients. |
| **PST** | Online when a valid `AZ:` or `TGA:` UDP reply has been received recently. |
| **Current** | Last azimuth received in an `AZ:xxx` reply. |
| **Target** | Last target received in a `TGA:xxx` reply. |
| **Last reply** | Local time of the most recent valid PstRotatorAz reply. |

### Choosing the Zeus listen IP

Use:

```text
127.0.0.1
```

when Zeus and ZeusPstBridge run on the same computer. This is the safest setting because other computers cannot connect to the TCP port.

Use the bridge computer's LAN address, for example:

```text
192.168.1.50
```

when Zeus runs on another computer and should connect only through that interface.

Use:

```text
0.0.0.0
```

only when the bridge must listen on all local IPv4 interfaces. Do not enter `0.0.0.0` as the destination address in Zeus; Zeus must connect to the bridge computer's real IP address.

### Port rules for multiple rotators

Every enabled channel must have a unique Zeus TCP endpoint.

Valid example:

```text
Rotator 1: 127.0.0.1 TCP 4533
Rotator 2: 127.0.0.1 TCP 4535
Rotator 3: 127.0.0.1 TCP 4537
```

Two PstRotatorAz instances on the **same computer** must use different UDP reply ports and should normally use separate command/reply pairs:

```text
PST instance 1: UDP 12000 commands, UDP 12001 replies
PST instance 2: UDP 12010 commands, UDP 12011 replies
PST instance 3: UDP 12020 commands, UDP 12021 replies
```

PstRotatorAz instances on **different IP addresses** may use the same UDP pair because ZeusPstBridge also identifies replies by source IP address:

```text
192.168.1.101: command 12000, reply 12001
192.168.1.102: command 12000, reply 12001
192.168.1.103: command 12000, reply 12001
```

Do not configure two channels with the same PstRotatorAz host and the same reply port. The bridge will reject this because incoming replies would be ambiguous.

## Configuring PstRotatorAz

Repeat these steps for every PstRotatorAz instance:

1. Open the UDP control settings in PstRotatorAz.
2. Enable UDP control.
3. Set the UDP command port, for example `12000`.
4. Confirm that query replies are sent on command port + 1, for example `12001`.
5. Ensure the selected PstRotatorAz instance is connected to and controlling the intended physical rotator.
6. Enter the PstRotatorAz computer's IP address and matching ports in the appropriate ZeusPstBridge row.
7. Start the bridge and press **Query Selected**.
8. Confirm that **PST** changes to **Online** and that **Current** shows the antenna azimuth.

The menu wording varies slightly between PstRotatorAz releases, but the required function is its UDP control port/interface.

The bridge sends commands such as:

```xml
<PST><AZIMUTH>85</AZIMUTH></PST>
<PST><STOP>1</STOP></PST>
<PST><PARK>1</PARK></PST>
<PST>AZ?</PST>
<PST>TGA?</PST>
```

Expected query replies are:

```text
AZ:85
TGA:85
```

A carriage return or line feed after the value is accepted.

## Configuring OpenHPSDR Zeus

Create or enable a Hamlib network/`rotctld` rotator in Zeus for each bridge channel.

For every Zeus rotator, enter:

- The IP address of the computer running ZeusPstBridge.
- The unique TCP port assigned to that channel.

When Zeus and the bridge run on the same PC:

```text
Rotator 1: host 127.0.0.1, port 4533
Rotator 2: host 127.0.0.1, port 4535
Rotator 3: host 127.0.0.1, port 4537
```

When Zeus runs on another PC and ZeusPstBridge runs at `192.168.1.50`:

```text
Rotator 1: host 192.168.1.50, port 4533
Rotator 2: host 192.168.1.50, port 4535
Rotator 3: host 192.168.1.50, port 4537
```

After Zeus connects, the channel's **Zeus** status changes from **Listening** to **1 client**. More than one program may connect to the same channel, in which case the client count increases.

ZeusPstBridge presents each channel as an azimuth-only rotator. Elevation is always reported as `0.000000`.

## Example installations

### Everything on one Windows PC

Use this when Zeus, ZeusPstBridge and one PstRotatorAz instance all run on the same computer.

```text
Bridge row:
  Zeus listen IP: 127.0.0.1
  TCP:            4533
  PST IP / host:  127.0.0.1
  PST port:       12000
  Reply:          12001

Zeus rotator:
  Host:           127.0.0.1
  Port:           4533
```

### Zeus and the bridge on one PC, remote PstRotatorAz computers

```text
Bridge PC: 192.168.1.50

Channel 1:
  Zeus: 127.0.0.1:4533
  PST:  192.168.1.101:12000, reply 12001

Channel 2:
  Zeus: 127.0.0.1:4535
  PST:  192.168.1.102:12000, reply 12001

Channel 3:
  Zeus: 127.0.0.1:4537
  PST:  192.168.1.103:12000, reply 12001
```

Because each PstRotatorAz computer has a different IP address, all three may use UDP 12000/12001.

### Zeus on a different PC from the bridge

Assume:

```text
Zeus PC:  192.168.1.40
Bridge PC: 192.168.1.50
PST PC:    192.168.1.101
```

Configure the bridge channel as:

```text
Zeus listen IP: 192.168.1.50
TCP:            4533
PST IP / host:  192.168.1.101
PST port:       12000
Reply:          12001
```

Configure Zeus as:

```text
Host: 192.168.1.50
Port: 4533
```

Allow inbound TCP 4533 and inbound UDP 12001 through the firewall on the bridge PC. Also allow the configured UDP command port through the firewall on the PstRotatorAz PC.

## Testing without a physical rotator

The `Tools` folder contains a simple PstRotatorAz UDP simulator and a Hamlib test client.

Do not run the simulator on a UDP port already being used by a real PstRotatorAz instance.

### 1. Start the simulator

Open PowerShell in the project folder and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\Tools\PstRotatorAz-Simulator.ps1 -CommandPort 12000
```

The simulator starts at 180 degrees and listens for the PstRotatorAz commands used by the bridge.

### 2. Start the bridge

Run the built `ZeusPstBridge.exe` and use the default first row:

```text
Zeus listen IP: 127.0.0.1
TCP:            4533
PST IP / host:  127.0.0.1
PST port:       12000
Reply:          12001
```

Press **Start Bridge**. The first row should become **PST Online** and display approximately `180.0°`.

### 3. Run the Hamlib test client

Open a second PowerShell window in the project folder:

```powershell
powershell -ExecutionPolicy Bypass -File .\Tools\Test-Rotctld.ps1 -Port 4533
```

The script:

1. Requests bridge capabilities.
2. Reads the current position.
3. Sends a target azimuth of 123 degrees.
4. Reads the updated position.
5. Sends Stop.

## Supported commands

ZeusPstBridge implements the core Hamlib commands needed for absolute azimuth control.

| Hamlib/rotctld input | Function | PstRotatorAz operation |
|---|---|---|
| `P 85 0` | Set position | `<PST><AZIMUTH>85</AZIMUTH></PST>` |
| `\set_pos 85 0` | Set position | `<PST><AZIMUTH>85</AZIMUTH></PST>` |
| `p` | Read position | Returns the most recently received `AZ:` value and elevation 0. |
| `\get_pos` | Read position | Returns the most recently received `AZ:` value and elevation 0. |
| `S` | Stop | `<PST><STOP>1</STOP></PST>` |
| `\stop` | Stop | `<PST><STOP>1</STOP></PST>` |
| `K` | Park | `<PST><PARK>1</PARK></PST>` |
| `\park` | Park | `<PST><PARK>1</PARK></PST>` |
| `_` / `\get_info` | Identification | Returns bridge information. |
| `dump_state` | Rotator limits | Returns the configured azimuth range and zero elevation range. |
| `1` / `dump_caps` | Capabilities | Reports position, stop and park support. |
| `q` | Disconnect | Closes that TCP client connection. |

Both ordinary and Hamlib extended response formats are supported.

Continuous clockwise/anticlockwise Hamlib `move` commands are intentionally reported as not implemented. The documented PstRotatorAz UDP interface used by this project does not provide a safe direct equivalent.

## Configuration and log files

The program stores per-user data in:

```text
%LOCALAPPDATA%\ZeusPstBridge
```

Normally this expands to a path similar to:

```text
C:\Users\YourName\AppData\Local\ZeusPstBridge
```

Files are:

```text
%LOCALAPPDATA%\ZeusPstBridge\config.json
%LOCALAPPDATA%\ZeusPstBridge\Logs\ZeusPstBridge-YYYY-MM-DD.log
```

Use **Open Config Folder** in the application to open this location.

If `config.json` is missing, the program creates a new default configuration. If the file cannot be parsed, the invalid file is copied to a timestamped `.bad-YYYYMMDD-HHMMSS` backup and a new default file is created.

A `sample-config.json` file is included with the source package.

### Advanced JSON-only settings

Some settings are retained in `config.json` but are not columns in the main grid:

| Property | Function |
|---|---|
| `OfflineTimeoutMs` | Time without a valid PST reply before the channel is marked Offline. |
| `QueryTarget` | Enables periodic `<PST>TGA?</PST>` target queries. |
| `SendOnAtStart` | Sends `<PST><ON>1</ON></PST>` when the channel starts. |
| `SendTrackAtStart` | Sends `<PST><TRACK>1</TRACK></PST>` when the channel starts. |

Close ZeusPstBridge before manually editing `config.json`. Keep a backup before making changes.

Example:

```json
{
  "Version": 1,
  "AutoStart": false,
  "LogProtocolTraffic": true,
  "Rotators": [
    {
      "Id": "11111111-1111-1111-1111-111111111111",
      "Enabled": true,
      "Name": "2 m",
      "ListenAddress": "127.0.0.1",
      "TcpPort": 4533,
      "PstHost": "192.168.1.101",
      "PstCommandPort": 12000,
      "PstReplyPort": 12001,
      "PollIntervalMs": 500,
      "OfflineTimeoutMs": 3000,
      "MinAzimuth": 0,
      "MaxAzimuth": 360,
      "QueryTarget": true,
      "SendOnAtStart": false,
      "SendTrackAtStart": false
    }
  ]
}
```

The application normalises the file back to 10 channel entries after loading and saving it.

## Firewall and network security

### Windows Firewall

Localhost connections using `127.0.0.1` do not normally require a firewall rule.

For remote Zeus connections, allow the configured Zeus TCP ports into the bridge PC. For example, run PowerShell as Administrator:

```powershell
New-NetFirewallRule `
  -DisplayName "ZeusPstBridge TCP 4533-4551" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 4533,4535,4537,4539,4541,4543,4545,4547,4549,4551 `
  -Action Allow `
  -Profile Private
```

For PstRotatorAz replies, allow the UDP reply ports required by the enabled channels. Example:

```powershell
New-NetFirewallRule `
  -DisplayName "ZeusPstBridge UDP replies" `
  -Direction Inbound `
  -Protocol UDP `
  -LocalPort 12001,12011,12021 `
  -Action Allow `
  -Profile Private
```

On each remote PstRotatorAz computer, allow its UDP command port or allow the PstRotatorAz application itself through the firewall.

Restrict rules to the Private profile and trusted LAN addresses wherever possible.

### Security warning

The Hamlib `rotctld` protocol used here has no authentication or encryption. Anyone able to connect to a channel's TCP port may be able to command that rotator.

- Prefer `127.0.0.1` when Zeus and the bridge share a PC.
- Keep the service on a trusted private LAN.
- Do not forward the TCP or UDP ports from an internet router.
- Do not expose the bridge directly to the public internet.
- Use firewall scope restrictions or a VPN for any remote-site use.

## Troubleshooting

### The bridge will not start

Read the message box and the log pane. Common causes are:

- No channel has **On** ticked.
- A TCP or UDP port is already in use.
- Two channels use the same Zeus listen IP and TCP port.
- Two channels point to the same PST host and reply port.
- The Zeus listen field is not an IPv4 address.
- The PstRotatorAz hostname cannot be resolved to an IPv4 address.
- A port is outside the valid range 1-65535.
- Poll interval is outside 200-10,000 ms.
- Minimum azimuth is not lower than maximum azimuth.

### Zeus status remains `Listening`

This means the channel started correctly but Zeus has not connected.

Check:

- Zeus is configured for a Hamlib network/`rotctld` rotator.
- Zeus is using the bridge computer's address, not the PstRotatorAz computer's address.
- Zeus is using the channel's TCP port.
- If Zeus is remote, the bridge is not listening only on `127.0.0.1`.
- The bridge PC firewall allows the TCP connection.
- No other `rotctld` service is using the same port.

### PST status remains `Offline`

This means no valid `AZ:` or `TGA:` reply has been received recently.

Check:

- PstRotatorAz UDP control is enabled.
- The bridge has the correct PstRotatorAz IP address.
- The command port is correct.
- The reply port is normally command port + 1.
- The bridge PC firewall allows the UDP reply port.
- The PstRotatorAz PC firewall allows incoming UDP commands.
- The two computers can ping each other where network policy permits.
- Another application is not already bound to the reply port.
- PstRotatorAz actually answers `<PST>AZ?</PST>` with `AZ:xxx`.

Enable **Protocol traffic log**, press **Query Selected**, and inspect the log. A transmitted `AZ?` with no following PST reply usually indicates a port, firewall or PstRotatorAz UDP configuration problem.

### Zeus connects but no position appears

The bridge does not invent an azimuth. Until a valid `AZ:` response has been received, a Hamlib position request returns a timeout error.

Confirm that the channel shows **PST Online** and a value in **Current** before testing position in Zeus.

### A commanded azimuth is rejected

The target must be between the channel's configured **Min AZ** and **Max AZ** values. This check applies both to Zeus commands and the manual Go button.

For a conventional rotator use `0` to `360`. For a rotator/controller that deliberately supports overlap, configure the appropriate accepted range, such as `-180` to `540`, only after verifying the physical system is safe to operate that way.

### Replies are reported as ambiguous

Two channels match the same PstRotatorAz source IP and UDP reply port. Assign a different UDP port pair to one of the PstRotatorAz instances.

### The program was moved but the old settings remain

Settings are not stored beside the executable. They are stored in:

```text
%LOCALAPPDATA%\ZeusPstBridge\config.json
```

Use **Open Config Folder** to locate or reset them.

### Finding unsupported Zeus commands

Enable **Protocol traffic log** and reproduce the problem. Unsupported Hamlib commands are written to the application log with the channel name. These logs are useful when extending compatibility with additional Zeus functions.

## Known limitations

- Version 0.1 is an azimuth-only bridge. Elevation is always returned as zero.
- Continuous Hamlib `move` commands are not translated.
- UDP provides no delivery acknowledgement. `RPRT 0` means the bridge accepted and transmitted a supported command; actual movement is confirmed only by later `AZ:` feedback.
- Park behaviour and the parked heading are controlled by PstRotatorAz, not by ZeusPstBridge.
- The program is not a complete implementation of every Hamlib `rotctld` command.
- There is no authentication or encryption on the Zeus TCP interface.
- Configuration is stored per Windows user.

## Safety

Rotators can move large antennas, dishes and mechanical structures without warning. Before enabling computer control:

- Verify physical end stops and controller limits.
- Set suitable Min AZ and Max AZ limits in the bridge.
- Keep people clear of the antenna and rotator structure.
- Confirm that every Zeus channel controls the intended PstRotatorAz instance.
- Test Stop Selected and STOP ALL before normal operation.
- Do not rely on software alone as the only emergency stop or travel-limit protection.

## Licence

ZeusPstBridge is supplied under the MIT Licence. See `LICENSE.txt`.

The software is provided without warranty. It is not affiliated with or endorsed by OpenHPSDR, Zeus, Hamlib or PstRotatorAz.
