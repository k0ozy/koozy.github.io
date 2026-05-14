---
title: "Chapter 1"
date: 2026-05-11T15:39:40-04:00
draft: false
tags: []
categories: []
ShowToc: true
TocOpen: false
---

Chapter 1 within Practical Malware Analysis focuses on basic malware static analysis techniques. Some of the techniques that are covered within this chapter include hashing, antivirus scanning, reviewing strings within the binary, detecting if the program is packed or not, reviewing imports/exports, and examining the various fields within the PE header. I will provide short answers to each question at the beginning and then walk through my methodology and go into more detail.

## Lab 1-1

### Questions

1. Upload the files to http://www.VirusTotal.com/ and view the reports. Does either file match any existing antivirus signatures?
	- Yes. Both files have multiple AV hits on VirusTotal
2. When were these files compiled?
	- **Lab01-01.dll** - 2010-12-19 08:16:38
	- **Lab01-01.exe** - 2010-12-19 08:16:19
3. Are there any indications that either of these files is packed or obfuscated? If so, what are these indicators?
	- No
4. Do any imports hint at what this malware does? If so, which imports are they?
	- The .exe and .dll file suggest that there are functions with backdooring capabilities, networking, and manipulating files on the system.
5. Are there any other files or host-based indicators that you could look for on infected systems?
	- Examine `Kerne132.dll`
6. What network-based indicators could be used to find this malware on infected machines?
	- IP Address `127.26.152.13`
7. What would you guess is the purpose of these files?
	- The .dll file is probably a backdoor of some kind and the .exe file is used to run the .dll

### Initial Triage

Lab 1-1 presents two different files: `Lab01-01.exe` and `Lab01-01.dll`. The first thing I want to confirm is if the files are what they say they are. Running `file` against both confirms that both are PE32 executables.
![File Command Output](lab1-1filecmd.png)

### Hashing and VirusTotal
After confirming the file type, I am going to hash them and then search VirusTotal for both hashes to answer the first question if either file match any existing antivirus signatures.
![File Hashes for Both Files](file-hash1.png)
	**Lab01-01.dll** [VirusTotal Report](https://www.virustotal.com/gui/file/f50e42c8dfaab649bde0398867e930b86c2a599e8db83b8260393082268f2dba) 
![AV Hits for Lab01-01.dll](vtdll1.png)
	**Lab01-01.exe** [VirusTotal Report](https://www.virustotal.com/gui/file/58898bd42c5bd3bf9b1389f0eee5b39cd59180e8370eb9ea838a0b327bd6fe47)
![AV Hits for Lab01-01.exe](vtexe1.png)

### Compilation Time
I will use Detect it Easy (DiE) to view the compilation time of both files. The compilation time will give us an idea of when the executable was created although it can be changed. Due to the compilation time, it is worth noting that both of these files might be related.
![Lab01-01.dll Compilation Time](diedll1.png)
![Lab01-01.exe Compliation Time](dieexe1.png)

### Packing and Obfuscation
A few different signals point to if a binary is packed or not.  
**Compiler/Linker info:** If DiE identifies a known compiler or linker, the binary is probably not packed.  
**Entropy:** Packed or encrypted data have higher entropy (~>7.0) than normal programs.   
**Virtual Size vs Raw Size:** If a section has a virtual size much larger than its raw size, it is likely that the binary is unpacking itself at runtime.  
**Strings:** Packed binaries tend to show fewer readable strings. An unpacked binary will show more in plain text. The strings output will be further down in the indicators section since it is not needed to show packing and obfuscation in this example.  
Die shows all of this so we can check each indicator without leaving the tool.  

**Lab01-01.dll**
![Linker/Compiler Information Lab01-01.dll](packeddll1.png)
![Entropy Information Lab01-01.dll](entropydll1.png)

**Lab01-01.exe**
![Linker/Compiler Information Lab01-01.exe](packedexe1.png)
![Entropy Information Lab01-01.exe](entropyexe1.png)

### Import Review
Reviewing the strings data gives an idea of what imports we will see in the files. All of the information can still be found within DiE but I am going to use PE-Bear this time to change it up. 
The Lab01-01.dll file also doesn't have any interesting imports in `MSVCRT.dll`. Within `KERNEL32.dll` there are functions such as `CreateProcess` and `Sleep` that suggest it includes some type of backdoor capability. Also within the `WS2_32.dll`, there are 10 functions that are imported by ordinal. One way to view what these functions would be would be to load the .dll within PE bear and find out what the ordinals map to. In this case, the ordinals map to networking functions.

The `Lab01-01.exe` file doesn't have any interesting imports in `MSVCRT.dll`, but within `KERNEL32.dll` there are functions for searching, opening and manipulating files. This suggests that the malware searches through the file system and that it can open and manipulate files.

**Lab01-01.dll**
![Imports Lab01-01.dll](importsdll1.png)
![Imports Lab01-01-2.exe](importsdll1-2.png)
![Imports Lab01-01-3.exe](importsdll1-3.png)

**Lab01-01.exe**
![Imports Lab01-01.exe](importsexe1.png)

### Indicators
If we review the strings from earlier after reviewing the imports there are a few things that stand out.  
**Lab01-01.dll**
The strings in this dll show an IP Address, as well as `exec` and `sleep` commands. It is likely that the `exec` command is used to run something with `CreateProcess`.
![Strings Lab01-01.dll](stringsdll1.png)  

**Lab01-01.exe**
There is a `Kerne132.dll` string that is made to look like `Kernel32.dll` as well as a .exe string. If this were going to be analyzed further, the `Kerne132.dll` file should be analyzed. Based off of the imports from the previous section it seems likely that the program is looking for .exe files on the host.
![Strings Lab01-01.exe](stringsexe1.png) 

---

## Lab 1-2
1. Upload the file to VT and view the reports. Does the file match any antivirus signals?
	- Yes. There are multiple AV hits on VirusTotal
2. Are there any indications that the file is packed or obfuscated? If it is, unpack it
	- Yes. The executable was packed using UPX.
3. Do any imports hint at the program's functionality?
	- The program creates a service and connects to the internet
4. What host or network based indicators could be used to identify this malware on infected machines?
	- MalService and hxxp://www.malwareanalysisbook[.]com

### Initial Triage
For this lab, the triage methodology remains the same so I won't rehash that for every lab. More detail will be provided if needed. The file was identified as a PE32 executable, hashed and uploaded to VirusTotal.  

![AV Hits for Lab01-02.exe](vtexelab1-2.png)
	**Lab01-02.exe** [VirusTotal Report](https://www.virustotal.com/gui/file/c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6)


### Packing and Obfuscation
Opening the file within DiE shows that the executable has been packed using UPX. UPX is a common packer and can be downloaded on [GitHub](https://github.com/upx/upx/releases/tag/v5.1.1). After downloading the file you can decompress it with `upx.exe -d [filename]`

![Linker/Compiler Information Lab01-02.exe](packedexe1-2.png)
![UPX Unpack](upxunpack1-2.png)

### Import Review and Indicators
There are a couple of interesting strings in this executable: `MalService` and `hxxp://www.malwareanalysisbook[.]com`.  The imports from `ADVAPI32.dll` tell us that this executable creates a service and the imports from `WININET.dll` show that it connects to the internet. When combining these, it is likely that `MalService` is the service that is created and the url from above is used with by `InternetOpenURL`.

![Strings Lab01-02.exe](stringslab1-2.png)
![Imports 1 Lab01-02.exe](impotslab1-2-1.png)
![Imports 2 Lab01-02.exe](impotslab1-2-2.png)



---
## Lab 1-3
1. Upload the file to VT and view the reports. Does the file match any antivirus signals?
	- Yes. There are multiple AV hits on VirusTotal
2. Are there any indications that this file is packed or obfuscated?
	- Yes. The raw size and virtual size are different, there are only `LoadLibraryA` and `GetProcAddress` imports, and PEiD shows that it was packed with FSG.
3. Do any imports hint at the program's functionality?
	- We cannot know until the file is unpacked
4. What host or network based indicators could be used to identify this malware on infected machines?
	- We cannot know until the file is unpacked

### Initial Triage

![AV Hits for Lab01-03](vtexe3.png)
	**Lab01-03.exe** [VirusTotal Report](https://www.virustotal.com/gui/file/7983a582939924c70e3da2da80fd3352ebc90de7b8c4c427d484ff4f050f0aec)

### Packing and Obfuscation
It also shows that there is some sort of packer that was used but it doesn't know what one. However, if we use PEiD it states that it is packed with FSG. 

![Strings Lab01-03.exe](stringsexe1-3.png)
![Linker/Compiler Information Lab01-03.exe](dieexe1-3.png)
![Linker/Compiler Information PEiD](peid1-3.png)


### Import Review and Indicators
Opening the file within DiE shows  `LoadLibraryA` and `GetProcAddress` are the only strings and imports that are seen. These two imports are needed for the packer's unpacking stub to dynamically resolve everything else from the OS after unpacking the import table into memory.

![Imports Lab01-03.exe](importslab1-3.png)


---
## Lab 1-4
1. Upload the file to VT and view the reports. Does the file match any antivirus signals?
	- Yes. There are multiple AV hits on VirusTotal
2. Are there any indications that the file is packed or obfuscated? If it is, unpack it
	- No
3. When was this program compiled?
	- 2019-08-30 15:26:59
4. Do any imports hint at the program's functionality?
	- This program appears to download a file from a website, loads data from the resource section of the PE file, writes to disk and executes the file.
5. What host or network based indicators could be used to identify this malware on infected machines?
	- `\system32\wupdmgr.exe` and `www[.]practicalmalwareanalysis[.]com/updater[.]exe`
6. Examine the resource in the resource section, what can we learn from this resource?
	- The executable is a downloader program that appears to download additional malware

### Initial Triage

![AV Hits for Lab01-04](vtexe4.png)
	**Lab01-04.exe** [VirusTotal Report](https://www.virustotal.com/gui/file/0fa1498340fca6c562cfa389ad3e93395f44c72fd128d7ba08579a69aaf3b126)

### Packing and Obfuscation
There is no indication that the program is packed. 

![Linker/Compiler Information Lab01-04.exe](packedexe1-4.png)


### Import Review and Indicators
There are some interesting strings that are found when reviewing the file. One being `www[.]practicalmalwareanalysis[.]com/updater[.]exe` which probably holds malicious code to be downloaded. It also includes `\system32\wupdmgr.exe`.   The imports from `advapi32.dll` suggests the program does something with permissions and the imports from `kernel32.dll` tells us the program loads data from the resource section, writes a file to disk, and executes the file. We can also see from the DiE image above, that there is a PE32 resource file in the resource section. The `GetWindowsDirectory` could be pointing to `\system32\wupdmgr` that was seen in the strings export. 

Resource Hacker can be used to save the file that was located in the resource section.

**Lab01-04.exe**
![Strings Lab01-04.exe](stringsexe1-4.png)
![Imports 1 Lab01-04.exe](importsexe1-3.png)
![Imports 2 Lab01-04.exe](importsexe1-3-2.png)
![ResourceHacker](reshacker.png)

**Extracted Bin**
![FileInfoBin](diebin.png)
![StringsBin](binstrings.png)

{{< figure src="" alt="" caption="" width="0" >}}
