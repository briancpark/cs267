# CS 267

CS 267 is Applications of Parallel Computers at UC Berkeley. I took this offering in Spring 2022 under Aydın Buluç and James Demmel. This repo contains my development setup and notes.

There is no source code here and the submodules are made private. Please DO NOT contact me for source code.

## Setup

The following setups are for Cori Supercomputer. This will retire on [January 17th, 2023](https://docs.nersc.gov/systems/cori/), so some of the setup will become or is irrelevant.

> Cori had its first users in 2015, and since then, NERSC's longest running system has been a valuable resource for thousands of users and projects. With the complete Perlmutter system scheduled to be operational for the 2023 allocation year, NERSC plans to decommision Cori at the end of the 2022 allocation year on January 17, 2023.

### Cori

Logging into Cori login nodes will require your password and 2FA. In order to avoid that and login in with just `ssh cori.nersc.gov`, you can use SSH proxy. Where `$USER` is your Cori user. Note that this will expire in 24 hours.

```sh
./sshproxy.sh -u bcp
```

For Cori, to activate a session on KNL nodes, you can do

```sh
salloc -N 1 -C knl -q interactive -t 01:00:00
```

Add this to your `~/.bashrc`:
```sh
alias interactive="salloc -N 1 -C knl -q interactive -t 01:00:00"
```

For reference, here is the CPU specs of the Intel KNL Xeon Phi that the KNL nodes use:
```
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              272
On-line CPU(s) list: 0-271
Thread(s) per core:  4
Core(s) per socket:  68
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               87
Model name:          Intel(R) Xeon Phi(TM) CPU 7250 @ 1.40GHz
Stepping:            1
CPU MHz:             1401.000
CPU max MHz:         1401.0000
CPU min MHz:         1000.0000
BogoMIPS:            2799.89
L1d cache:           32K
L1i cache:           32K
L2 cache:            1024K
NUMA node0 CPU(s):   0-271
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl est tm2 ssse3 fma cx16 xtpr pdcm sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch ring3mwait cpuid_fault epb pti intel_ppin ibrs ibpb fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms avx512f rdseed adx avx512pf avx512er avx512cd xsaveopt dtherm ida arat pln pts
```

With the information above, the theoretical peak performance of the KNL nodes is 3 TFLOPS per node (double precision).

There are GPU allocations available

```sh
salloc -C gpu -t 60 -c 10 -G 1 -q interactive -A mp309 
```

### Bridges-2
For use of GPUs, we used the Bridges-2 GPU nodes.

The GPU nodes use a DGX-1 system, each equipped with 8 V100 32GB

This makes the theoretical peak of V100 7.8 TFLOPS (double precision).

```sh
salloc -N 1 -p GPU-shared --gres=gpu:1 -q interactive -t 01:00:00
```

For allocations of more than 1 GPU to use a multi-GPU setup connected via NVLink, you can use:

```sh
salloc -N 1 -p GPU --gres=gpu:8 -q interactive -t 01:00:00
```

For adding an alias, add this to your `~/.bashrc`:

```sh
alias interactive="salloc -N 1 -p GPU-shared --gres=gpu:1 -q interactive -t 01:00:00"
```


When ever executing code that utilizes NVLink connections via NCCL, make sure to have this environment variable set, or else code will fail to communicate between GPUs:

```sh
export "NCCL_LAUNCH_MODE"=PARALLEL
```

Profiling is pretty useful and easy with CUDA programs. You can easily do so by:

```sh
nsys profile --stats=true -t nvtx,cuda <code> <args>
```

If you ever want to check the status of NVLink connections, you can do:

```sh
nvidia-smi nvlink --status -i 0
```