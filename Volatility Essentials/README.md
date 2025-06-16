![Banner](https://github.com/user-attachments/assets/d7f03c2d-4636-4fea-890c-4502b807bb27)


# Volatility Essentials

## Task 2 : Volatility Overview
1. Volatility is a cross-platform, open-source memory forensics framework.
2. It is modular and extensible, widely used for analyzing RAM dumps.
3. The latest version is Volatility 3, which:
   - Uses dynamic symbol resolution (no need for static OS profiles).
   - Supports newer OS versions and memory layouts.
   - Provides complete insight into the runtime system state.

### Architectural Components
> **Memory Layers:**

  - Represent the hierarchy of memory address spaces (from raw to virtual).

>  **Symbol Tables:**

  - Use OS-specific debugging symbols to interpret kernel and process structures.

>  **Plugins:**

  - Modular tools that extract forensic artifacts using memory layers and symbol tables.




### System Requirements & Installation
> Requirements:

  - Python 3.6+

> Libraries:

  - pefile, capstone, yara-python (for PE analysis, disassembly, and YARA scanning).

> Installation Steps:

  
  Firstly, we need to download the setup..
   
    git clone https://github.com/volatilityfoundation/volatility3.git
    cd volatility3
    
Volatility 3 is ready to run directly from source after cloning.
    
    python3 vol.py -h

The -h flag shows the available command-line options.

<pre>
Volatility 3 Framework 2.26.2
usage: vol.py [-h] [-c CONFIG] [--parallelism [{processes,threads,off}]] [-e EXTEND] [-p PLUGIN_DIRS] [-s SYMBOL_DIRS] [-v] [-l LOG] [-o OUTPUT_DIR] [-q]
              [-r RENDERER] [-f FILE] [--write-config] [--save-config SAVE_CONFIG] [--clear-cache] [--cache-path CACHE_PATH] [--offline | -u URL]
              [--filters FILTERS] [--hide-columns [HIDE_COLUMNS ...]] [--single-location SINGLE_LOCATION] [--stackers [STACKERS ...]]
              [--single-swap-locations [SINGLE_SWAP_LOCATIONS ...]]
              PLUGIN ...

An open-source memory forensics framework

options:
  -h, --help            Show this help message and exit, for specific plugin options use 'vol.py  --help'
  -c CONFIG, --config CONFIG
                        Load the configuration from a json file
------TRUNCATED--------
  
</pre>


<br>

## Task 3 : Memory Acquisition and Analysis

  Memory acquisition is a foundational step in forensics that must be performed in a manner that ensures we maintain the integrity of evidence. The process and the deployment environment used vary from one OS to another.


1. For `Windows` systems, the following tools can be used to conduct memory acquisition:
    - **DumpIt:** captures a full physical memory image on 32/64‑bit Windows and automatically hashes the output.
    - **WinPmem:** Open‑source driver‑based tool that acquires RAM in RAW/ELF formats and embeds acquisition metadata for chain‑of‑custody.
    - **Magnet RAM Capture:** GUI‑driven collector that snapshots volatile memory on live Windows hosts while minimising footprint.
    - **FTK Imager:** The most common commercial tool that acquires memory and selected logical artefacts alongside disk imaging functions. 


2. For `Linux` and `macOS` systems, we can employ the services of the following tools:
    - **AVML:** Lightweight Microsoft CLI utility that dumps Linux memory to a compressed ELF file without requiring a kernel module.
    - **LiME:** Loadable Kernel Module for Linux that captures full volatile memory over disk or network and supports ARM/x86 architectures.
    - **OSXPmem:** macOS‑specific fork of Pmem that creates raw memory images on Intel‑based Macs for subsequent Volatility analysis.



<br>

### System Information Enumeration
  I used the `windows.info` plugin for this task.  

    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.info
 
![image](https://github.com/user-attachments/assets/93e4a20e-dd26-4e20-af33-42aac4cef1fe)

> Findings:
  
  -  **Build Version:** `2600.xpsp.080413-2111`
  -  **Memory Acquisition Time:** `2012-07-22 02:45:08` UTC



## Task 4 : Listing Processes and Connections


### Process Analysis

  Firstly, I used `windows.pslist` plugin to list the running processes in the memory dump. The output from this plugin will include all current and terminated processes and their exit times.

    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.pslist
    
  So, according to the output I identify the process id of the _`Adobe`_ i.e. `1640` and it's process name was `reader_sl.exe`. So after this I used another process realted plugin i.e. `windows.pstree`. This plugin help with analysis of complete story of the processes and what may have occurred at the extraction time.
Also, I used the `grep` to get the specific only result related to our process..
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.pstree | grep "reader_sl.exe"

![image](https://github.com/user-attachments/assets/662bf7de-d357-466b-a9f6-7adca59752c9)



<br>
  By looking into the result of the command we ran earlier i.e. 
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.pslist

![image](https://github.com/user-attachments/assets/041b77c2-2f38-4061-84dd-a5f413bc7419)
  
  We can find the `PID`(Process ID) and `PPID` (Parent Process ID) for the processes.. So, we can match the PPID and PID of the `reader_sl.exe` and `explorer.exe`
  
> Findings:

  -  **Suspicious Process:** `reader_sl.exe` (PID 1640)
  -  **Parent Process:** `explorer.exe` (PID 1484)
  -  **Binary Path:** `C:\Program Files\Adobe\Reader 9.0\Reader\reader_sl.exe` 

<br>

 ### DLL

  We will be using the plugin named  `windows.dlllist`. This plugin will list all DLLs associated with processes at extraction time. To simplify the output, I added filers like `--pid` and `grep`.
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.dlllist --pid 1640 | grep -v -i "system32"

![image](https://github.com/user-attachments/assets/9b77db1f-8860-47d2-892d-2f87fe298c7e)
    So, we can see in the output there are `4` files are listed (i.e. MSVCP80.dll,comctl32.dll.. ) out of which `3` are the dll's 

> Findings:

- **Non-System32 DLLs:** `3`

<br>

### Handles Analysis
  
  In this we can use the plugin  `windows.handles` to look into the details and handles of files and threads from a host. To simplify the output, I added filers like `--pid` and `grep`.

    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.handles --pid 1640 | grep "KeyedEvent"
    
![image](https://github.com/user-attachments/assets/9c7a37ba-68d1-4e49-b44e-e90acae61bd9)

> Findings:

- **KeyedEvent Handle:** `CritSecOutOfMemoryEvent`



<br>

## Task 5 : Volatility Hunting and Detection Capabilities

  We will be using plugin named `windows.malfind`. This plugin will attempt to detect injected processes and their PIDs along with the offset address and the infected area's Hex, Ascii, and Disassembly views. 
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.malfind
    
![image](https://github.com/user-attachments/assets/82b8d586-53e3-4ed1-99e4-b1349cc4e81c)

  So, I looked into the `magic bytes` of the header. And  we can identify the magic bytes in header of the processes named `explorer.exe` and `reader_sl.exe` are **4D 5A** which is little endian representation of the hexadecimal header for `PE`(portable executable) file. 
  
![image](https://github.com/user-attachments/assets/509dcec1-b0a1-44c0-840f-519b30233c51)

> Findings:

- Processes with `MZ` Headers: `explorer.exe, reader_sl.exe` (indicates code injection)

<br>

## Task 6 : Advanced Memory Forensics

  Volatility 3’s `windows.ssdt` plugin enables analysts to inspect this table for any irregularities. So, we will use this plugin along with filters like `grep` for clean output..
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-1.vmem windows.ssdt | grep -E 'NtCreateFile' | column -t
    
![image](https://github.com/user-attachments/assets/c6a4589e-d878-458b-a21e-7373082a36fa)

  So, looking into the output we can see our address of `NtCreateFile` system call

> Findings:

- Address of `NtCreateFile` call: `0x8056e27c`

<br>


## Task 7 : Practical Investigations

We can use [Cheat Sheat](https://hackmd.io/@TuX-/BymMpKd0s) for the finding plguin and identifying their uses..


1 Firstly, I used the `pslist` plugin along specifying the process ID i.e. 740.

      python3 vol.py -f ~/Desktop/Investigations/Investigation-2.raw windows.pslist --pid 740
    
![image](https://github.com/user-attachments/assets/a72f4f7a-c496-4784-ad4f-5c1da20d19a0)
  
  Looking into the output, the suspicious process name is `@WanaDecryptor@`


<br>

**2** To find the path of the executable, I used the `dlllist` plugin. Along with the filter like `--pid` and `grep` for clear output. 
  
    python3 vol.py -f ~/Desktop/Investigations/Investigation-2.raw windows.pslist --pid 740 | grep -E '@WanaDecryptor@' | column -t
  
![image](https://github.com/user-attachments/assets/ec0cce12-b4da-46b5-b880-ba3b22ac4dd6)

  Looking into the output we can identify the path for _`@WanaDecryptor@`_ file. i.e. `C:\Intel\ivecuqmanpnirkt615\@WanaDecryptor@.exe`

<br>

**3** To identify the Parent process, I used the pstree plugin. At first, I run it with `grep` specifying the PID (Process ID) for the exceutable i.e. `740` 

      python3 vol.py -f ~/Desktop/Investigations/Investigation-2.raw windows.pstree | grep "740"

   By looking  in the respone of it, I found the PPID (Parent Process ID) i.e. `1940`. So I again run the command changing the PID in the grep to `1940`. 
      
      python3 vol.py -f ~/Desktop/Investigations/Investigation-2.raw windows.pstree | grep "1940"
      
![image](https://github.com/user-attachments/assets/fb92a631-faff-4f6b-89f1-b86d2afa1c1a)

   So , after these two steps I got the name of the parent process.. i.e. `tasksche.exe`


<br>

**4**  To identify the malware in our system we can simply judge by the name of the `.exe` (i.e. `WanaDecryptor`) that it is the `Wannacry` malware. But considering a situation where name is not properly shown, we can download the dump from the memory files.. for this we can use the `dumpfiles` plugin. For this we need to give the path of the existing directory where we will be saving the dump files. I also added the `PID` to specify the downloading instead of downloading all the dump files.

      python3 vol.py -f ~/Desktop/Investigations/Investigation-2.raw -o ~/Desktop/output/ windows.dumpfiles --pid 740

![image](https://github.com/user-attachments/assets/cc5c0305-93f5-4b31-974d-fe842aadf5c0)

   After, I downloaded the dumpfiles, I simple found the hash of the `.exe` file using `md5sum`.

      md5sum <filename.exe>
![image](https://github.com/user-attachments/assets/951fc4cf-2624-4658-95dd-8f02f3ec9e4f)

   After this, I pasted the hash into the [Virustotal](https://www.virustotal.com/gui/home/search)
   
![image](https://github.com/user-attachments/assets/8ac4c293-40e5-499e-b608-a5273f21177e)

   So, we can see Family labels ( `wanna`, `wannacryptor` , `wannacry`)

<br>

**5** To identify all files loaded from the malware working directory, we can use the `windows.filescan` plugin.

![image](https://github.com/user-attachments/assets/57f48507-70f7-4224-8a25-7e75e49d389c)


> Findings:

- **Malicious Process:** `@WanaDecryptor@.exe` (PID 740)
- **Binary Path:** `C:\Intel\ivecuqmanpnirkt615\@WanaDecryptor@.exe`
- **Parent Process:** `tasksche.exe` (PID 1940)
- **MD5 Hash:** `c523cdfa774ddabfb3dc47f9ed945698`
- **Malware:** `Wannacry`

<br>

## General Volatility Workflow

![image](https://github.com/user-attachments/assets/0c6f4e76-38ba-4c69-9c85-b16b14dedd28)

- **Profile Identification:** `windows.info`
- **Process Analysis:** `pslist`, `pstree`, `psscan`
- **Malware Detection:** `malfind`, `vadinfo`
- **File Extraction:** `dumpfiles`, `pedump`
- **Network Analysis:** `netscan`, `netstat`
- **Registry Forensics:** `hivelist`, `printkey`

> Final Notes:
- Always verify hashes with `VirusTotal`.
- Use `yarascan` for custom rule detection.
- Check for unlinked drivers (driverscan) in `rootkit` cases.






