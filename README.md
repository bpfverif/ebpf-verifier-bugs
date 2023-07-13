# ebpf-verifier-bugs
This repository is a collection of bugs in the eBPF verifier's range analysis discovered by our tool Agni ([CAV'23 research paper](https://people.cs.rutgers.edu/~sn624/papers/agni-cav23.pdf), [CAV'23 artifact](https://github.com/bpfverif/ebpf-range-analysis-verification-cav23)). Our tool produces several proof-of-concept (POC) eBPF programs that manifest the bugs found by static analysis of the eBPF verifier source code from the Linux kernel. Here, we demonstrate how to run the programs in Linux and witness the manifestation of the bug. 

## Structure of this repository

Each top level directory is a kernel version, and contains the POC eBPF programs (.txt) synthesized by Agni for the particular kernel version. Agni uses the following naming convention: `bpf_prog_<KernelVersion>_<Instruction>_<Domain>.txt`. For example, file titled `5.8_jlt_s64.txt` contains an eBPF program that manifests a bug in kernel version `5.8`, on the `jlt` (jump-less-than) instruction, and demonstrates a violation of the soundness in the `s64` (signed-64) abstract domain. In addition to eBPF instructions/macros, Agni also inserts some comments into the POC eBPF program: metadata useful for the `bpf_test_tool`. 

**bpf_test_tool.** The `bpf_test_tool` submodule contains scripts to quickly compile and run these examples in a Linux environment. It consists of a C wrapper around our POC eBPF programs that invokes the verifier. The script `bpf_test.sh` does the compilation, running, and printing out the verifier's log. Usually, the verifier log is very long; our script truncates this log to only the useful parts.

## Getting started
To get started, you will need to compile and run the kernel version for which you want to test the bugs. Skyzkaller has a quick [guide](https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md) to compile a custom kernel and run it in QEMU. The rest of this guide assumes you have Linux v5.8.0 (commit bcf87687) running. 

![Alt text](images/image1.png)

Our tool Agni has synthesized several POC eBPF programs (67 for kernel v5.8). All of these programs are present in the `5.8/` directory. Here, we demonstrate 3 of the verifier bugs found by Agni, and how to run them.

## Triggering verifier bugs

First clone this repository and the `bpf_test_tool` submodule.
```
git clone --recurse-submodules https://github.com/bpfverif/ebpf-verifier-bugs.git
```

Enter the `bpf_test_tool` directory
```
cd bpf_test_tool
```

### Example 1: An ALU instruction (a known CVE in BPF_AND)

First, let's look at a known [CVE](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3490), that was a bug in the abstract analysis of the `BPF_AND` instruction. This bug has been [exploited](https://web.archive.org/web/20221020081759/https://www.graplsecurity.com/post/kernel-pwning-with-ebpf-a-love-story#toc-1) to show local privilege escalation. Agni is able to produce a POC eBPF program that manifests the vulnerability **_automatically_** and with **_fewer instructions_**.

 Let's load the POC eBPF program `bpf_prog_5.8_and_u32.txt`. The script `bpf_test.sh` is useful.

```
./bpf_test.sh ../5.8/bpf_prog_5.8_and_u32.txt
```

This should produce the following output. 

![Alt text](images/image5.png)

**Reading the output.** The script only prints out lines `9` and `10` from the verifier log, omitting the other lines. (These "relevant" line numbers were inserted as comments into `bpf_prog_5.8_and_u32.txt` by Agni.) Line `9` corresponds to the instruction involving the `BPF_AND` instruction: `r1 &= r3`. Line `10` consists of the verifier's beliefs in the values of the registers immediately after executing the instruction (more precisely, the abstract domain values used to track the values in the registers). Finally, after the second set of dashed lines, the _actual_ values in 10 eBPF registers (`r0` through `r9`) are printed. 

**The bug.** Note that the filename `bpf_prog_5.8_and_u32.txt` mentiones the abstract domain that this program violates soundness in: `u32`, so we have to look for `u32_min_value`, `u32_max_value` in the verifier log. When we look at register `R2`'s abstract domain values, the error in the analysis of `BPF_AND` in the `unsigned 32` domain is clear: `u32_min_value` is greater than `u32_max_value`. This should never be the case in a sound analysis.

### Example 2: A jump instruction

```
./bpf_test.sh ../5.8/bpf_prog_5.8_jge_s32.txt
```
![Alt text](images/image4.png)

**Reading the output.** In case of a jump instruciton, the verifier tracks register values along both the true (go-to) and false (fall-through) paths. In this particular example, the verifier error takes place along the fall-through path. (Agni encodes this information as comments in `bpf_prog_5.8_jge_s32.txt`.) Our script prints only the relevant parts of the verifier log: instruction `6` (the jump instruction) and the verifier's beliefs about the values of the registers in the fall-through case. 

**The bug.** From the filename, `bpf_prog_5.8_jge_s32.txt` we know to look at the `signed 32` abstract domain in the verifier log: `s32_min_value` and `s32_max_value`. Register `R2`'s values are interesting: `s32_min_value=-2147483646,s32_max_value=-1`. Looking at the latter part of the output, the _actual_ value in register `R2` is `0x80000000252f502e`, i.e. `623857710` when interpreted as a signed 32-bit integer. It is clear that this number is not contained within the `signed 32` bounds. This is a bug in the verifier's analysis of `BPF_JGE`.

### Example 3: Another jump instruction

```
./bpf_test.sh ../5.8/bpf_prog_5.8_jlt_s64.txt
```
![Alt text](images/image3.png)

**Reading the output.** This time, our script prints out only the parts of the verifier log corresponding to the case of the branch instruction where the jump indeed occurs (which happens on line `9`).

**The bug.** Let's look at the `signed 64` domains of all the registers, `R2` in particular. We see that `smin_value=-9223372034707292160`, the minimum possible value in the register according to the verifier. But we also see that the _actual_ value in `R2` is `-9223372036720558079`, which is lesser than `smin_value`. Again, this is a bug in the verifier's analysis of `BPF_JLT`.

