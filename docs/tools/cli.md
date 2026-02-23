# CanaryBit Command-Line Interface (CLI)

*Manage CanaryBit resources with interactive commands*

---

The CanaryBit Command-Line Interface (CLI) is a cross-platform command-line tool to connect to your CanaryBit account and execute commands on CanaryBit resources.

## Install 

You can install the CanaryBit CLI locally on Linux, macOS, or Windows computers. 

Download the binary from CanaryBit's S3 public bucket:

```
$ curl -fsSL https://canarybit-public-binaries.s3.eu-west-1.amazonaws.com/[VERSION]/[OS_TARGET] -o cb
```

where `[VERSION]` is 

- `0.2.5` (latest) 

and `[OS_TARGET]` is one of the following:

- Linux: `cb-x86_64-unknown-linux-gnu`
- Windows: `cb-x86_64-pc-windows-msvc.exe`
- Apple Intel: `cb-x86_64-apple-darwin`
- Apple M-series: `cb-aarch64-apple-darwin`

**Example (Linux)**
```
$ curl -fsSL https://canarybit-public-binaries.s3.eu-west-1.amazonaws.com/0.2.5/cb-x86_64-unknown-linux-gnu -o cb
```

### Current Version

For information about the current (latest) release, see the release notes. To find your installed version run:

```
cb --version
```

## Authentication

CanaryBit CLI allows you to authenticate and fetch credentials for specific CanaryBit services:

```
cb login
```

and display your CanaryBit account information:

```
cb account
```

## Data collection

CanaryBit CLI does not collect data by default. 