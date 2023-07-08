# ebpf-verifier-bugs
This repository is a collection of bugs in the eBPF verifier's range analysis discovered by our tool [Agni](https://github.com/bpfverif/ebpf-range-analysis-verification-cav23). Our tool produces proof-of-concept eBPF programs that manifest the bugs in the eBPF verifier. Here, we demonstrate how to run the programs in Linux and witness the manifestation of the bug. 

## This repository

- Each top level directory is a kernel version, and contains the eBPF programs (.txt) synthesized by Agni for the particular kernel version. 
- The `bpf_test_tool` submodule contains scripts to quickly compile and run these examples in a Linux environment. 




