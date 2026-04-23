# CanaryBit Command-Line Interface (CLI)

*Manage CanaryBit resources with interactive commands*

---

The CanaryBit Command-Line Interface (CLI) is a cross-platform command-line tool to connect to your CanaryBit account and execute commands on CanaryBit resources.

## Requirements

- A [CanaryBit Account](https://auth.confidentialcloud.io/signup?client_id=54g4h9tpulnnkmhivgn5nipjki&redirect_uri=https%3A%2F%2Finspector.confidentialcloud.io%2F&response_type=code&code_challenge_method=S256&code_challenge=ngK8ZsXEC3A72nogaAmpKpR_LnB5kvCqOvr8z6qDWZI&scope=openid+profile+email);

## Install 

Download the binary from CanaryBit's S3 public bucket:

```
$ curl -fsSL https://canarybit-public-binaries.s3.eu-west-1.amazonaws.com/[VERSION]/[OS_TARGET] -o cb
```

where `[VERSION]` is the `cb` version (e.g. `0.3.0`) and `[OS_TARGET]` is one of the following reference architectures:

- Linux: `cb-x86_64-unknown-linux-gnu`
- Windows: `cb-x86_64-pc-windows-msvc.exe`
- Apple Intel: `cb-x86_64-apple-darwin`
- Apple M-series: `cb-aarch64-apple-darwin`

**Example**
```
$ curl -fsSL https://canarybit-public-binaries.s3.eu-west-1.amazonaws.com/0.2.5/cb-x86_64-unknown-linux-gnu -o cb
```

## Subcommands

### Authentication

Authenticate towards CanaryBit Identity Provider and fetch a valid token for specific CanaryBit services.

Source your credentials:

```
export CB_USERNAME=***
export CB_PASSWORD=***
```

and run:

```
cb login
```

### Account

Display your CanaryBit account information.

```
cb account
```


### Assets

Get access to CanaryBit assets such as `cbclient` agent, Tower Premium and more.

```title="List"
./cb list cbclient
```

```title="Download"
./cb download cbclient 0.2.6/cbclient
```

### Current Version

For information about the current (latest) release, see the release notes. To find your installed version run:

```
cb --version
```

### Help

Print the help of the given subcommand(s).

```
$ cb --help
```

## Data collection

CanaryBit CLI does not collect data by default. 