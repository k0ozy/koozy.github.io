---
title: "Windows Processes"
date: 2026-05-11T22:22:11-04:00
draft: false
tags: []
categories: []
ShowToc: true
TocOpen: false
---
#### Process Threads
- Windows processes are made up of 1 or more threads running concurrently
- Thread is a set of instructions that can be executed independently in a process
- Threads can communicate and share data

#### Process Memory
- Memory is allocated to a process when it is created and the allocation amount can be set by the process itself. 
- Uses both physical and virtual memory

#### Memory Types
- Private memory - Dedicated to a single process and cannot be shared. Used to store data that is specific to the process
- Mapped memory - Can be shared between 2 or more processes. Used to share data between processes (shared libraries, memory segments, files). It is visible to other processes but cannot be modified by other processes
- Image Memory - Contains the code and data of executable file. Used to store the code and data used by the process. Usually related to DLL files loaded into a process's address space

### Process Environment Block (PEB)
- Data structure that contains information about a process
- It is used by the OS to store information about processes as they are running and is used by the Windows loader to launch applications
- Every Process created has its own PEB structure
```c
typedef struct _PEB {
  BYTE                          Reserved1[2];
  BYTE                          BeingDebugged;
  BYTE                          Reserved2[1];
  PVOID                         Reserved3[2];
  PPEB_LDR_DATA                 Ldr;
  PRTL_USER_PROCESS_PARAMETERS  ProcessParameters;
  PVOID                         Reserved4[3];
  PVOID                         AtlThunkSListPtr;
  PVOID                         Reserved5;
  ULONG                         Reserved6;
  PVOID                         Reserved7;
  ULONG                         Reserved8;
  ULONG                         AtlThunkSListPtr32;
  PVOID                         Reserved9[45];
  BYTE                          Reserved10[96];
  PPS_POST_PROCESS_INIT_ROUTINE PostProcessInitRoutine;
  BYTE                          Reserved11[128];
  PVOID                         Reserved12[1];
  ULONG                         SessionId;
} PEB, *PPEB;
```
- ##### BeingDebugged 
	- Flag that indicates if the process is being debugged or not 1 - True 0 - False
- ##### Ldr
	- Pointer to `PEB_LDR_DATA` structure in the PEB. Contains information about process's loaded DLL modules. Includes a list of Dlls, base address and size of each module
```c
typedef struct _PEB_LDR_DATA {
  BYTE       Reserved1[8];
  PVOID      Reserved2[3];
  LIST_ENTRY InMemoryOrderModuleList;
} PEB_LDR_DATA, *PPEB_LDR_DATA;
```
- ##### ProcessParameters
	- Contains the commandline parameters passed to the procces when created
	- Pointer to RTL_USER_PROCESS_PARAMETERS struct
```c
typedef struct _RTL_USER_PROCESS_PARAMETERS {
  BYTE           Reserved1[16];
  PVOID          Reserved2[10];
  UNICODE_STRING ImagePathName;
  UNICODE_STRING CommandLine;
} RTL_USER_PROCESS_PARAMETERS, *PRTL_USER_PROCESS_PARAMETERS;
```
- ##### AtlThunkSListPtr & AtlThunkSListPtr32
	- Used by the Active Template Library module to store a pointer to a linked list of thunking functions
	- Thunking functions are used to call functions that are implemented in a different address space. Often represent functions exported from a DLL
- ##### PostProcessInitRoutine
	- Used to store a pointer to a function that is called by the OS after TLS (Thread Local Storage) initialization has been completed for all threads in a process. 
- ##### SessionId
	- Unique Identifier assigned to a single session. Used to track the activity of the user during the session

### Thread Environment Block (TEB)
- Data structure that stores information about a thread. Environment, security context, etc
- Stored in the thread's stack and used by windows kernel to manage threads
```c
typedef struct _TEB {
  PVOID Reserved1[12];
  PPEB  ProcessEnvironmentBlock;
  PVOID Reserved2[399];
  BYTE  Reserved3[1952];
  PVOID TlsSlots[64];
  BYTE  Reserved4[8];
  PVOID Reserved5[26];
  PVOID ReservedForOle;
  PVOID Reserved6[4];
  PVOID TlsExpansionSlots;
} TEB, *PTEB;
```
- ##### PEB
	- Pointer to the PEB structure. PEB is located in the TEB and is used to store info about currently running processes
- ##### TlsSlots
	- TLS slots are locations in the TEB that are used to store thread-specific data. 
	- Each thread in windows has its own TEB and each TEB has a set of TLS slots
	- Apps can use the slots to store data like variables, handles, states that are specific to the thread
- ##### TlsExpansionSlots
	- Set of pointers used to store thread-local data for a thread.
	- Reserved for use by system DLLs

### Process and Thread Handles
- Identifiers (PID or Unique thread ID) can be used to open a handle using 
	- OpenProcess
	- OpenThread
- Handles should always be closed to avoid handle leaking with CloseHandle
