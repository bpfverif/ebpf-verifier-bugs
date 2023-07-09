# ebpf-verifier-bugs
This repository is a collection of bugs in the eBPF verifier's range analysis discovered by our tool [Agni](https://github.com/bpfverif/ebpf-range-analysis-verification-cav23). Our tool produces several proof-of-concept (POC) eBPF programs that manifest the bugs in the eBPF verifier. Here, we demonstrate how to run the programs in Linux and witness the manifestation of the bug. 

## This repository

Each top level directory is a kernel version, and contains the POC eBPF programs (.txt) synthesized by Agni for the particular kernel version. The naming convention used for these programs is:

```
bpf_prog_<KernelVersion>_<Instruction>_<Domain>.txt
```

For example, a POC eBPF program titled `5.8_jlt_s64.txt` contains an eBPF program that manifests a bug in kernel version `5.8`, on the `jlt` (jump-less-than) instruction, and demonstrates a violation of the soundness of the abstract interpretation in the `s64` (signed-64) abstract domain

The `bpf_test_tool` submodule contains scripts to quickly compile and run these examples in a Linux environment. It consists of a C wrapper around our POC eBPF programs that invokes the verifier. The script `bpf_test.sh` does the compilation, running, and printing out relevant parts of the verifier log. 

The rest of the readme discusses a few example bugs found by Agni, how to run them inside a live Linux kernel running in QEMU, and also explains the bugs themselves. 

To get started, you will need to compile and run the kernel version for which you want to test the bugs. Skyzkaller has a quick [guide](https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md) to compile a custom kernel and run it in QEMU

The rest of this guide assumes you have Linux v5.8 (commit bcf87687) running. 

## Recreating verifier bugs

First clone this repository and update the `bpf_test_tool` submodule.
```
git submodule init
git submodule update
```

### Example 1: 5.8_sub_s64

### Example 2: 5.8_jlt_s64

### Example 3: 5.8_jge_s32
