---
title: Run test on CI
sidebar_position: 4
---

## Prerequisites

- you **[DOWNLOAD](ttps://github.com/parimatch-tech/PMetriumNative/tree/main/PackageRegistry)** or build a single executable file (plus some additional files which come together) for PMetrium Native of the target OS, for more details please follow this **[link](./00-prepare-workstation.md#ii-run-as-a-single-file-application-localhostci)**
- CI Runner have a network access to the InfluxDB host
- you have up and run Grafana Dashboard for metrics visualization 
- you have a CI runner with the physical connection to real android devices or emulators and ready Android Device Bridge (adb) 
	:::caution
	PMetrium Native does not provide an out-of-the-box device farm ready for performance tests as this farm looks almost the same as for functional tests. So here we expect that on CI you already have such a runner for functional tests
	:::
- optional: runner has available `curl` on CLI

## CI Runner settings

1. Copy a .tar.gz archive with single executable file for PMetrium Native (plus some additional files which come together) to the runner Workstation. Example with the `curl`:
	```bash
	> curl \
		-o PMetriumNative.win-x86.v1.0.0.tar.gz \
		"https://github.com/parimatch-tech/PMetriumNative/tree/main/PackageRegistry/PMetriumNative.win-x86.v1.0.0.tar.gz"
	```

	Where:
	- `win-x86` - one of the target OS architectures. Available: 
		- `win-x86`, `win-x64`, `win-arm`, `win-arm64`, `osx-x64`, `osx-arm64`, `linux-x64`, `linux-arm`
	- `v1.0.0` - version of the ready build package, see **[the full list](https://github.com/parimatch-tech/PMetriumNative/tree/main/PackageRegistry)**
2. Extract archive, open folder and run PMetrium Native server as a separate process on the runner. You may add some additional settings to it throught **[PMetrium Native config](../architecture/03-development/03-pmetrium-config.md)**. For example:
	```bash
	> tar -xzf PMetriumNative.win-x86.v1.0.0.tar.gz --directory ./PMetriumNative.win-x86.v1.0.0
	> cd ./PMetriumNative.win-x86.v1.0.0
	> ./PMetrium.Native.Host &
	```
	& - move the process to the background on Linux-based runners
3. You may add PMetrium Native server to the startup of the runner OS
4. Execute health check of the PMetrium Native:
	```bash
	> curl http://localhost:7777/HealthCheck
	```
	The response should be just `OK` if everything is fine

## Run performance test
Now you just need to call PMetrium Native endpoint before functional test to start measurement and one endpoint after the test is finished, example for the CI:

```bash
> curl -G -d "device=192.168.0.103:5555" -d "app=com.parimatch.ukraine" -d "cpuApp=no" http://localhost:7777/Start
> dotnet test ./src/PMetrium.Native/FunctionalTests  --filter ColdStart
> curl -G -d "device=192.168.0.103:5555" http://localhost:7777/Stop
```

Please note that command `dotnet test ./src/PMetrium.Native/FunctionalTests  --filter ColdStart` is just an example here for the functional test

:::important
**device** and **app** are required parameters for PMetrium Native server, see **[PMetrium Native API](../architecture/03-development/04-pmetrium-api.md)**
:::

:::tip
You may handle the interaction with the PMetrium Native server from inside of the functional tests framework, in that case, it would be much easier to run tests, see the **[example](./02-run-localhost.md#run-from-code)**
:::