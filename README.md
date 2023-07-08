# ebpf-verifier-bugs
This repository is a collection of bugs in the eBPF verifier's range analysis discovered by our tool [Agni](https://github.com/bpfverif/ebpf-range-analysis-verification-cav23). Our tool produces proof-of-concept eBPF programs that manifest the bugs in the eBPF verifier. Here, we demonstrate how to run the programs in Linux and witness the manifestation of the bug. 

## This repository

Each top level directory is a kernel version, and contains the eBPF programs (.txt) synthesized by Agni for the particular kernel version. 

The `bpf_test_tool` submodule contains scripts to quickly compile and run these examples in a Linux environment. 

The rest of the readme discusses a few example bugs found by Agni, how to run them inside a live Linux kernel running in QEMU, and also explains the bugs themselves. 

To get started, you will need to compile and run the kernel version for which you want to test the bugs. Skyzkaller has a quick [guide](https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md) to compile a custom kernel and run it in QEMU

The rest of this guide assumes you have Linux v5.8 (commit bcf87687) running. 

