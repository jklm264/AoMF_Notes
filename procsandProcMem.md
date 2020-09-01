# Processes and Process Memory

## Processes In Memory

Every Linux proc has a `task_struct` in kernel memory. Contains:

- open file descriptors
- memory maps
- authentication credentials
- and more

`Task_structs`'s are allocated in the kernel's memory cache (`kmem_cache`), specifically in `task_struct_cachep` . Can be found as the global variable with the same name.

- *tasks* - the procs' reference into a linked list of active procs
- *mm* - stores memory management data like the DTB value (offset of the procs' directory table base; *mm -> pgd*). **For kernel threads, this value is NULL.**
- *pid*
- *parent* - PPID. If procs' parent exits, the child is then claimed by `init`
- *children*
- *cred* - stores UID, GID, and stores credential information for the proc.

- *comm* - name of executable, or kthread, in 16B char array. For kthreads, will ends with "/<number>" to indicate CPU where thread is executed.
- *start_time*



## Enumerating Processes

Back-end allocators for storing `task_struct`'s when not in `kmem_cache` are *SLAB* and *SLUB* depending on kernel configuration options.

- Serve same purpose as *SLAB* allocators on Windows and the *SLAB allocator* on Mac OS X
- **Purpose**: To (de)allocate structures of the same size in an efficient manner from a preallocated block of kernel memory.



*SLAB* used to work by tracking allocations of all objects of a particular type. Now, they (Linux) use *SLUB* which does not track allocations - making them unreliable for enumerating objects.



Of note, *SLAB* is still widely used on Android.

[The research (**also by Andrew Case**)](http://dfir.org/research/DFRWS-2010-kmem_cache.pdf)



Since kernel cache isn't reliable, there are two other sources which are reliable for enumerating processes: 1) Active process list and 2) PID hash table.

### Active Process List

> "Contrary to popular belief, this list is not actually exported to userland. Thus, most live response and system administration tools do not reference it to enumerate processes. "

Using `vol.py linux_pslist` will enumerate processes by walking the active process list pointed to by the global *init_task* variable. This variable is statically allocated within the kernel, initialized at boot, has a PID of 0, and is named *swapper* (recall from previous section "swapper" is the initial Linux process in physical memory). 

<kbd>**Note**, *swapper* is not listed when `$ps` is run.</kbd>

The output of `$vol.py linux_pslist` will notably denote kernel threads without a DTB value.

#### Linking Procs to Users

To link procs to users, you can cross-reference the UIDs and GIDs with the contents from `/etc/passwd` and `/etc/group`

Example user *apache2* is part of *www-data* group (33):

```bash
$grep 33 /{passwd,group}
/etc/passwd:www-data:x:33:33:www-data:/var/www:/bin/sh 
/etc/group:www-data:x:33:Parent Child Relations
```

My own example to find which users are part of the `rtkit` group:

```bash
$grep 126 /etc/{passwd,group}
/etc/passwd:rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
/etc/group:rtkit:x:126:
```

#### Parent and Child Relations

There's `$vol.py linux_pstree` for this.

Of note, everything is under *init* except for kthreads. Some malware my hide by enclosing their names in brackets [process_name] to blend in as a kernel thread; as kthreads are visualized as kernel with surrounding brackets in this view as well as `$ps` and `$top`.

### PID Hash Table

Directories under `/proc` are populated from the global PID hash table.

`$ps` will parse this directory, and malware authors will figure out ways to avoid their proc being logged here. This is done by tampering with the data structures **OR** performing control flow redirection from within the `/proc` fs or its supporting system calls.



## Process Address Space

When loading an executable, the kernel will create data structures to track shared libraries, stack, heap, and other regions in the process address space. The kernel will track a proc's start/end address, permissions, backing file info, and metadata (for caching and searching).



`task_struct.mm` is a type of `mm_struct`

- *mmap and mm_rb* store proc memory mappings as a linked list and red-black tree.
- *pgd* is address of the process' DTB. **THIS** is what populates the DTB column of `$vol.py linux_pslist`!
- *owner* is a pointer to the *task_struct* that owns the current *mm_struct*. This can be an alternative source of proc listings since *mm_struct*'s are tracked by the cache.

### Enumerating Proc Maps

(slide 644)

Going back to the `mm_struct`, *mmap* and *mm_rb* are of importance to us.

- *mmap* is a linked list of  `vm_area_struct`
- *mm_rb* which stores the same `vm_area_struct` but as red-black trees. This is so the kernel can quickly find mappings during page faults, or when a new memory range needs to be allocated. Nodes are sorted by starting address of each region.



`dt("vm_area_struct")` contains

- *vm_flags* indicate whether region was mapped RWX
- *vm_file* points to the `file` structure of the file the region maps (NULL if a memory-backed region)



The OS uses that *mmap* member to populate the `/proc/<pid>/maps` directory. `$vol.py linux_proc_maps` uses this directory (which walks that *task_struct.mm.mmap* list). It can reveal memory ranges of stack, heap, permissions, inode number, and more per proc. This is useful to find signs of code injection (can see full paths of `*.so`'s)

### Recovering Sections of Memory

Example: extract executable section of the *init* binary.

1. Find address: `$vol.py linux_proc_maps | grep init`
2. Extract entire init executable section:`$vol.py linux_dump_map -p 1 -s 0x400000 -D dump` 
3. Check contents:`$file dump/taslk.1.0x400000.vma`

### Analyzing Command-line Arguments

As previously shown, `vol.py linux_pslist` compiles names of running procs from each proc's *task_struct.comm* member. Recall this only grabs the first 16B, and does not include path or arguments. To find this additional info we look to `vol.py linux_psaux`.

- **Note:** *psaux* works by switching to the procs' address space with *task_struct.get_process_space() and then reading the address value of *mm_struct.arg_start*.



**In summary,** 

- linux_pslist is useful for address ranges, DTB value, GID, and offset
- linux_psaux is useful for argument list and full path

### Manipulating Command-Line Args

Malware can manipulate the output of `$ps` by overwriting command-line args. In the kernel source code (`fs/proc/base.c`) we see the declaration of the `/proc/<pid>/cmdline` file which calls *proc_pid_cmdline* which does the following:

` res = access_process_vm(task, mm->arg_start, buffer, (mm->arg_end - mm->arg_start), 0);`

- Recall in C, arg starts at argv[0] which is the program name!



#### To Exploit

```c
#$gcc baddie.c -o backdoor
#include <stdio.h>
int main(int argc, char *argv[])
{
	char *my_args = "nginx\x00-k\x00start\x00"; #Nullbyte makes sure args are included in program name
	memcpy(argv[0], my_args, 17);
	while(1)
		sleep(1000);
}
```

##### Testing

For whatever reason, in order for proc to display arg values of *my_args*, need to compile binary twice  over.

1. `$gcc baddie.c -o backdoor`
2. `$gcc baddie.c -o backdoor 2`
3. `$./backdoor2 arg1 &`

##### Results

Only show obfuscated name: `$cat /proc/<pid>/cmdline` and `$ps aux | grep <pid>` 

Show *backdoor* name: $`cat /proc/<pid>/{comm,stat,maps}`



**In summary**, make sure to check both pslist and psaux, as pslist might find out more than you think. Normal `$ps` uses the *cmdline* member which can be overwritten, whereas the *comm* and *stat* members cannot be!



## Process Environment Variables

A process' initial set of env vars are stored in a statically allocated buffer of null-terminated strings. The kernel will always keep track of these in *mm_struct.env_start/end*.

- Use `vol.py linux_psenv`



**Major Notes:** 

- Kernel threads don't have env vars!

- *OLDPWD* is the directory that the user was in before changing to the current directory

- You can tell if a proc was spawned over SSH because they will include SSH-esq env vars like *SSH_CONNECTION* and *SSH_CLIENT*. Also *USER* indicates the user who logged in over SSH.

- "**_**" variable (the underscore) tells you the full path of the command executed

  

## Open File Handles

"Everything is a file" -Linux



`dt("file")` contains

- *f_path* holds a reference to info needed to reconstruct the name and path
- *f_mode* tells whether file was opened for RWX
- *f_pos* tells where next RW will occur
- *f_mapping* points to where file's contents are on disk
- *f_op* shows a set of operations pointers (really functions) that are called when a proc RW or seeks. Rootkits hook these to hide files on live machines.



File Descriptors (fd) are stored in kernel memory. Each proc has a table (like Windows; see [chapter 7 of AoMF: Processes](pht.md#Processes)).

- NULL pointer means the fd is not in use.
- To find a proc's file descriptor table see *task_struct.files* of type *files_struct*.

- Recall  0 (stdin), 1 (stdout), and 2 (stderr)
- fd are usually all set to /dev/null in network applications



- Use `vol.py linux_lsof` to walk a proc's fd table to print total number and full path.

### Keyloggers

As an example: [Google's klk](https://github.com/kernc/logkeys/blob/master/src/logkeys.cc)

1. linux_pslist to find keylogger name
2. linux_psaux to find full cmdline (or comm). See [Manipulating Command Line Arguments](#Manipulating-Command-Line-Args)
3. linux_lsof to see where input, output, and errors (and other fd) are being (re)directed to.



## Saved  Context State

Edwin Smulder created Linux vol.py plugins for enumerating active threads and their execution context. He created plugins to grab process info during context switch including its threads, kernel stack, system calls, and its stack frames.

Found [here](https://www.volatilityfoundation.org/2013-c19yz)



## Bash Memory Analysis

`dt("_hist_entry")` which represents a line of a *.bash_history* file, contains

- *line* is the actually command entered
- *timestamp*, beginning with a `#` and stored as epoch time

### Bash History

Bash normally logs commands into `~/.bash_history`.

- Ways to circumvent commands being logged
  1. History file variable (*HISTFILE*) can be unset or just point it to `/dev/null`. Ex: `$unset HISTFILE`
  2. Set history size variable (*HISTSIZE*) to 0
  3. When SSH, use the *-T* to prevent pseudo-terminal allocation.



- Use `vol.py linux_bash` uses the *_hist_entry* structs. Actually it scans the heap for the `#` prefixed to each *_hist_entry.timestamp*. The plugin will then rescan the heap for any pointers to a `#` char.

### Bash Cmd Hash Table

Bash also keeps a hashtable with the full path to the commands AND number of times they executed: On live system just type `$hash`.



- Use `vol.py linux_bash_hash`

#### The Fake rm Cmd

This fake rm will not delete any file prefixed with v01.

To detect this, we can use `vol.py linux_bash_hash` to view the full path of that rm file. We can also check the path (via `vol.py linux_bash_env`) variable; though, this is less helpful here since the path that has the fake binary could still be for legit purposes.



## See for Later

- dt("task_struct.cred"); credntials in processes? Like password-protected? [See here](https://blog.cubieserver.de/2018/modify-process-credentials-in-linux-kernel/) :: [what are they (Types of Credentials)]([https://www.kernel.org/doc/Documentation/security/credentials.txt#:~:text=In%20Linux%2C%20all%20of%20a,'cred'%20in%20its%20task_struct.](https://www.kernel.org/doc/Documentation/security/credentials.txt#:~:text=In Linux%2C all of a,'cred' in its task_struct.))























