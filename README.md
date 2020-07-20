
# event-generator
> Generate a variety of suspect actions that are detected by Falco rulesets.

[![Release](https://img.shields.io/github/release/falcosecurity/event-generator.svg?style=flat-square)](https://github.com/falcosecurity/event-generator/releases/latest)
[![License](https://img.shields.io/github/license/falcosecurity/event-generator?style=flat-square)](LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/falcosecurity/event-generator?style=flat-square)](https://goreportcard.com/report/github.com/falcosecurity/event-generator)
[![Docker pulls](https://img.shields.io/docker/pulls/falcosecurity/event-generator?style=flat-square)](https://hub.docker.com/r/falcosecurity/event-generator)


**Warning** — We strongly recommend that you run the program within Docker (see below), since some commands might alter your system. 
    For example, some actions modify files and directories below /bin, /etc, /dev, etc.
    Make sure you fully understand what is the purpose of this tool before running any action.

## Usage

The full command line documentation is [here](./docs/event-generator.md).

### List actions

```shell
$ event-generator list

helper.ExecLs
helper.NetworkActivity
helper.RunShell
k8saudit.ClusterRoleWithPodExecCreated
k8saudit.ClusterRoleWithWildcardCreated
k8saudit.ClusterRoleWithWritePrivilegesCreated
k8saudit.CreateDisallowedPod
k8saudit.CreateHostNetworkPod
k8saudit.CreateModifyConfigmapWithPrivateCredentials
k8saudit.CreateNodePortService
k8saudit.CreatePrivilegedPod
k8saudit.CreateSensitiveMountPod
k8saudit.K8SConfigMapCreated
k8saudit.K8SDeploymentCreated
k8saudit.K8SServiceCreated
k8saudit.K8SServiceaccountCreated
syscall.ChangeThreadNamespace
syscall.CreateFilesBelowDev
syscall.DbProgramSpawnedProcess
syscall.MkdirBinaryDirs
syscall.ModifyBinaryDirs
syscall.NonSudoSetuid
syscall.ReadSensitiveFileTrustedAfterStartup
syscall.ReadSensitiveFileUntrusted
syscall.RunShellUntrusted
syscall.ScheduleCronJobs
syscall.SystemProcsNetworkActivity
syscall.SystemUserInteractive
syscall.UserMgmtBinaries
syscall.WriteBelowBinaryDir
syscall.WriteBelowEtc
syscall.WriteBelowRpmDatabase
```

### Run actions
```
event-generator run [regexp]
```
Without arguments it runs all actions, otherwise only those actions matching the given regular expression.

For example, to run `syscall.MkdirBinaryDirs` and
`syscall.ModifyBinaryDirs` actions only:
```shell
$ sudo event-generator run syscall\.\*BinaryDirs

INFO sleep for 1s                                  action=syscall.MkdirBinaryDirs
INFO writing to /bin/directory-created-by-event-generator  action=syscall.MkdirBinaryDirs
INFO sleep for 1s                                  action=syscall.ModifyBinaryDirs
INFO modifying /bin/true to /bin/true.event-generator and back  action=syscall.ModifyBinaryDirs
```

Useful options:
- `--loop` to run actions in a loop
- `--sleep` to set the length of time to wait before running an action (default to `1s`)

All other options are documented [here](./docs/event-generator_run.md).


#### With Docker

Run all events with the Docker image locally:

```shell
docker run -it --rm falcosecurity/event-generator run
```


#### With Kubernetes

Run the following command to create the Service Account (`falco-event-generator`), Cluster Role, and Role that will allow the tool to create objects in the current namespace:

```shell
kubectl apply -f deployment/role-rolebinding-serviceaccount.yaml
```

Run all events once using a Kubernetes job:

```shell
kubectl apply -f deployment/run-as-job.yaml
```

Run all events in a loop using a Kubernetes deployment:

```
kubectl apply -f deployment/event-generator.yaml
```

**N.B.**
The above commands apply to the `default` namespace. Use the `--namespace` option to use a different namespace. Events will be generated in the same namespace.

## Collections

### Generate System Call activity
The `syscall` collection performs a variety of suspect actions that are detected by the [default Falco ruleset](https://github.com/falcosecurity/falco/blob/master/rules/falco_rules.yaml).

```shell
$ docker run -it --rm falcosecurity/event-generator run syscall --loop
```

The above command loops forever, incessantly generating a sample event each second. 


### Generate activity for the k8s audit rules
The `k8saudit` collection generates activity that matches the [k8s audit event ruleset](https://github.com/falcosecurity/falco/blob/master/rules/k8s_audit_rules.yaml).


```shell
$ event-generator run k8saudit --loop --namespace `falco-eg-sandbox`
```
> N.B.: the namespace must exist already.

The above command loops forever, creating resources in the `falco-eg-sandbox` namespace and deleting the after each iteration.

**N.B.**
- the namespace must already exist
- to produce any effect the Kubernetes audit log must be enabled, see [here](https://falco.org/docs/event-sources/kubernetes-audit/)


## Test rules

Since `v0.4.0`, this tool introduces a convenient integration test suite for Falco rules. Basically the `event-generator test` command can run actions and test them against a running Falco instance.

> This feature requires Falco 0.24.0 or newer. Before using the command below, you need [Falco installed](https://falco.org/docs/installation/) and running with the [gRPC Output](https://falco.org/docs/grpc/) enabled.

#### Test locally (`syscall` only)

Run the following command to test `syscall` actions on a local Falco instance (connects via Unix socket to `/var/run/falco.sock` by default):

```shell
sudo ./event-generator test syscall
```

#### Test on Kubernetes

Then, run the following command to create the Service Account (`falco-event-generator`), Cluster Role, and Role that will allow the tool to create objects in the current namespace:

```shell
kubectl apply -f deployment/role-rolebinding-serviceaccount.yaml
```

Finally:

```shell
kubectl apply -f deployment/run-test.yaml
```

Note that to test `k8saudit` events, you need [Kubernetes audit log] enabled both in Kubernetes and Falco.

## FAQ

### What sample events can be generated by this tool?
See the [events registry](https://github.com/falcosecurity/event-generator/tree/master/events).

### Can I contribute by adding new events?
Sure! 

Check out the [events registry](https://github.com/falcosecurity/event-generator/tree/master/events) conventions, then feel free to open a PR.

Your contribution is highly appreciated.

### Can I use this project as a library?
This project provides three main packages that can be imported and used separately:

- `/cmd` contains the CLI implementation
- `/events` contains the events registry
- `/pkg/runner` contains the actions runner implementations

Feel free to use them as you like on your projects.

## Acknowledgments

Special thanks to Mark Stemm (**@mstemm**) — the author of the [first event generator](https://github.com/falcosecurity/falco/tree/2126616529e7015ff88653b7491dc1937d7e54e5/docker/event-generator).
