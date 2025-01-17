# Artifact Evaluation for "TAROT:A CXL SmartNIC-Based Defense Against Multi-bit Errors by Row Hammers Attacks", ASPLOS 2024.
This repository contains artifacts and workflows for reproducing the experiments presented in the ASPLOS 2024 paper by C. Song et al.

[![DOI](https://zenodo.org/badge/767037552.svg)](https://zenodo.org/doi/10.5281/zenodo.10867231)

# Contents
1. Reproducing "RH-induced Bit Flips". (related Fig. 4, 5, 6, 7, 9 and 13)
2. Reproducing "Performance evaluation". (related Table 3, 4, Fig. 10, 14 and 15)

# Hardware pre-requisities
Intel(R) Xeon(R) CPU (Code name Haswell or Broadwell)

We has been successfully tested on:
- Intel(R) Xeon(R) CPU E5-2640 v3 @ 2.60GHz (Haswell)
- Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz (Broadwell)

For reverse engineering the mapping function of physical memory to DRAM addresses, it is highly recommended to install one DIMM of DRAM in the slot.

# Software pre-requisities

g++ >= 8,  cmake (>= 3.14),
python (>= 3.0) for Post data process.

# 1. Reproducing "RH-induced Bit Flips".

1) Allocate N * 1GB hugepages on boot time. (e.g., 4 * 1GB)

   - Update grub file
   ```  
   $ sudo vi /etc/default/grub
   ```
   
   ```
   GRUB_CMDLINE_LINUX_DEFAULT="default_hugepagesz=1G hugepagesz=1G hugepages=4"
   ```
   
   ```
   $ sudo update-grub
   ```

   - Mount hugetlbfs

   ```  
   $ sudo vi /etc/fstab
   ```
   
   ```
   none /mnt/huge hugetlbfs pagesize=1G,size=4G 0 0
   ```

   ```  
   $ sudo reboot
   ```

2) Reverse-engineering Physical Address to DRAM Address Mapping
   We use "DRAMA" to reverse engineering the Physical Address to DRAM Address Mapping.
   
   https://github.com/IAIK/drama.git
   (Peter Pessl et al. "DRAMA: Exploiting DRAM Addressing for Cross-CPU Attacks". In 25th USENIX Security Symposium (USENIX Security 16).)

   We provide the mapping function for Haswell and Broadwell server in the modified code of row hammer attacks.

3) Running RH attack program.

   We modiied "BLACKSMITH" Row-hammer fuzzer for N-sided Row-hammer attack with stripe-pattern.

   https://github.com/comsec-group/blacksmith
   (Patrick Jattke et al. "BLACKSMITH: Scalable Rowhammering in the Frequency Domain".)

   - update RH attack code
     
   ```  
   $ git clone https://github.com/comsec-group/blacksmith.git
   $ cd blacksmith
   $ git apply ../1_RH_BIT_FLIP/RH_ATTACK_PATCH/TAROT_mod.patch
   ```


   - Before you run the RH attack program, update Physical to Dram mapping function and supported_cpus

   ```  
   $ vi ./src/Memory/DRAMAddr.cpp

   Update "struct MemConfiguration"
   We updae the mapping function w/ Haswell and Broadwell cpus. We tested on the "E5-2640", "E5-2680"
   ```

   ```  
   $ vi ./src/Blacksmith.cpp

   update supported_cpus list

     std::vector<std::string> supported_cpus = {
         ...
         "E5-2640",
         "E5-2680",
         " your cpu model in "lscpu" "
     };
   ```
  
  
   
   - run RH attack program
     
   ```  
   $ mkdir build
   $ cd build
   $ cmake ..
   $ make -j$(nproc)
   // For 1 rank DIMM
   $ sudo ./blacksmith --dimm-id 1 --runtime-limit 259200000 --ranks 1 -a 150
   // For 2 rank DIMM
   $ sudo ./blacksmith --dimm-id 1 --runtime-limit 259200000 --ranks 2 -a 150
   ```

5) Post data processing.

   We provide python code and example result files for post data processing.
   Refer "./1_RH_BIT_FLIP/POST_PROCESSING/README.md"

6) Trouble shooting.

   - System Crash by UE.
    
     When Uncorrectable Error is generated, system will be crashed.
     For monitoring the UEs, disable ecc.
     
     1) ECC off on the bios menu.
     2) ECC off by modifying MSR register using PCM (please refer ./1_RH_BIT_FLIP/troubleshoot/README.md).

   - No RH-error from RH attack program.
    
     Since latest DRAMs have in-dram mitigation logic, it is difficult to reproduce bit-flip.
     Modifying REFRESH rate of DRAM helps to generate bit-flip errors.
     Please refer ./1_RH_BIT_FLIP/troubleshoot/README.md


# 2. Reproducing "Performance evaluation".
   
Please refer to [2_PERF_EVAL/README.me](https://github.com/chihuns2/ae-asplos2024-TAROT/blob/main/2_PERF_EVAL/README.md)


   
   
