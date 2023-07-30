+++
author = "psykomal"
title = "The FS - File System"
date = "2023-07-21"
description = ""
tags = [
    "12-days", "file", "database"
]
+++






I have been meaning to learn about database internals. But before that, I wanted to understand the simplest database - the file. Doing this would help me document a mental model of the filesystem.

> As we’ve discussed before, mental models are what you are really trying to develop when learning about systems. For file systems, your mental model should eventually include answers to questions like: what on-disk structures store the file system’s data and metadata? What happens when a process opens a file? Which on-disk structures are accessed during a read or write? By working on and improving your mental model, you develop an abstract understanding of what is going on, instead of just trying to understand the specifics of some file-system code (though that is also useful, of course!).
>        ~ From OSTEP Ch 40 - [File System Implementation](https://pages.cs.wisc.edu/~remzi/OSTEP/file-implementation.pdf)


OS is all about abstractions. The process is an abstraction of the CPU and Memory (via Address Spaces). Thread is an abstraction over something schedulable/runnable. The file system is an abstraction over persistent storage. 


## The Basics

- **File** - A file is a container for an array or stream of bytes.
	- Each file has a user-given name, inode number. 
	- The inode of the file is a data structure that contains file-specific metadata, and more specially the location of data blocks of the file where the actual data is stored.
- **Directory** - A directory structure is very similar to a file. It is in fact a file whose data is the contents or files and child directories inside the directory 
- **File Descriptor** - A per-process number to identify a file. Each process maintains an array of file descriptors, each of which refers to an entry in the system-wide **open file table**. Each entry in this table tracks which underlying file the descriptor refers to, the current offset, and other relevant details such as whether the file is readable or writable.


Here are some key aspects of files from the OS perspective:

1. **Data Container**: A file serves as a container for storing data, regardless of its type. It can hold anything from text and images to program code and multimedia content.
2. **Metadata**: Each file has associated metadata, which includes information like the file name, file size, creation date, modification date, file permissions, and more. This metadata helps the OS manage and organize files efficiently.
3. **File System**: The OS manages files through a file system, which is a hierarchical structure that organizes files in directories or folders. The file system provides a way to locate, access, and organize files on storage devices, such as hard drives or SSDs.
4. **File Operations**: The OS provides a set of operations to work with files, including creating, opening, reading, writing, closing, renaming, moving, and deleting files. These operations are essential for interacting with the data stored in files.
5. **File Access Control**: The OS enforces file access control through permissions, determining which users or processes can read, write, or execute specific files. This helps protect sensitive data and maintain system security.
6. **File Extensions**: In many operating systems, files are identified by their names and extensions, which indicate the file type. For example, ".txt" for text files, ".jpg" for image files, and ".mp3" for audio files. File extensions help the OS associate the correct application to open a file when a user interacts with it.
7. **Stream I/O**: Files can be accessed through stream input/output (I/O) operations, allowing data to be read from or written to a file sequentially or randomly.
8. **Virtual File Systems**: Modern operating systems often use a virtual file system layer to provide a unified interface for accessing different types of storage devices and file systems. This abstraction allows the OS to support various storage technologies transparently.


### File I/O Operations


```bash
$ strace cat foo

...
read(3, "helo\n", 131072)               = 5
write(1, "helo\n", 5helo
)                   = 5
read(3, "", 131072)                     = 0
munmap(0x7fe3b475c000, 139264)          = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
...

```

- `open`, `read`, `write`, `link`, `unlink` are some of the system calls provided by the kernel for I/O
	- `  int open(const char *pathname, int flags, mode_t mode);`
- `read` function - ` ssize_t read(int fd, void *buf, size_t count);`
- `read` is sequential. To access a file from some offset, the OS offers `lseek`
	- `   off_t lseek(int fd, off_t offset, int whence);`
	- Note: `lseek` does not mean a physical disk seek operation. The OS abstracts that for us. 


### Relating to Processes

- Calling `open` returns a file descriptor which is a reference to the file. This information is maintained by the **open file table** by the OS.
- When `fork()` is called, the child processes share the file descriptors of the parent
- `dup` is used to create a new file descriptor pointing to the same file. Useful in the case of pipes

{{< figure
		  src="file_imgs/file_descr.png"
		  caption="From OSTEP Ch 39"
>}}


### How editors like Vim edit files

- Editing files is tricky because if the disk fails or crashes in between the editing, the file can get into an inconsistent state and be unrecoverable
- Editors use an intelligent way to overcome this by making use of an atomic command - `rename`
- `rename` is implemented as an atomic call which means if the system crashes, the file will either be in the new or old state but not in between
	- ` int rename(const char *oldpath, const char *newpath);`
- `write()` does not write to the disk immediately. It buffers in memory to write to disk in batches to make things efficient. `fsync` is used to force write to disk. It responds by forcing all dirty (i.e., not yet written) data to disk
- This is what the editing looks like. The editor creates a tmp copy of the original file and edits that file. On save, it calls `fsync` (commit the changes to the disk) and then renames that file to its original name. Since rename is atomic, no harm is done to the original file in any way.

```C
int fd = open("foo.txt.tmp", O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR); 
write(fd, buffer, size); // write out new version of file 
fsync(fd); 
close(fd); 
rename("foo.txt.tmp", "foo.txt");
```

------


# The Mental Model


{{< figure
		  src="file_imgs/blocks.png"
		  caption="From OSTEP Ch 40"
>}}


- The disk is divided into equal blocks or pages of 4KB
- Let's assume there are 65 blocks as layed out above
- The above blocks are 0 referenced
- Blocks \[8-64) are for data
- Blocks \[3-8) are for Inode information
- Block 1 if inode bitmap and Blcok 2 is data bitmap
	- These bitmaps are for tracking the free blocks
- Block 0 is the superblock. Contains info about the filesystem which is the first block the OS will read for loading the file system info


{{< figure
		  src="file_imgs/inode.png"
		  caption="From OSTEP Ch 40"
>}}


- There are 80 inodes possible in this model. 

{{< figure
		  src="file_imgs/ext2_inode.png"
		  caption="From OSTEP Ch 40"
>}}


- Now the inodes contain pointers to the data block. There are 3 ways here
	- The inode itself contains all the pointers to all the data blocks
	- Extent-based - An extent is a list of tuples like (pointer, num of contiguous blocks). This is compact and saves space however is not as flexible as first approach. Extra care should be taken to preallocate contiguous blocks when creating blocks. 
	- **Linked List** - The inode contains a pointer to the first data block. The first d-block contains a pointer to the second and it's a linked list of data blocks. However, accessing a block means many disk I/Os which is inefficient. A workaround is to have the linked list stored in a separate single block and can be read just once. This approach often less prominent is still used in systems like FAT (File Allocation Table). Old Windows used this approach. 

### The Multi-Level Index

- To support bigger files, file system designers have had to introduce different structures within inodes. One common idea is to have a special pointer known as an indirect pointer. Instead of pointing to a block that contains user data, it points to a block that contains more pointers, each of which points to user data. Thus, an inode may have some fixed number of direct pointers (e.g., 12), and a single indirect pointer. If a file grows large enough, an indirect block is allocated (from the data-block region of the disk), and the inode’s slot for an indirect pointer is set to point to it. Assuming 4-KB blocks and 4-byte disk addresses, that adds another 1024 pointers; the file can grow to be (12 + 1024) · 4K or 4144KB.
- Adding a double or triple indirect pointer is not uncommon with this approach to support GBs and TBs of file data

{{< figure
		  src="file_imgs/4LfZB.png"
		  caption="Multi-level index"
>}}


### Directory, Links

- Now you can see how a directory is not much different than a file
- The directory inode is stored with the inode sections and the data file stores tuples of the file name and the inode pointers 

Let's take some time here to talk about hard and soft links
- Hard link  (`ln foo foo-hard`) creates a file with the same inode
- Soft link creates a file that contains the path of the old file. This doesn't even have blocks but stores the path in the inode itself. If the path is too long to fit in the inode, it then creates blocks
- Now when we call `rm`, the OS calls `unlink` and need not remove the underlying data yet. It's because multiple files could be holding the inode which is indicated by the refcnt in the inode. Only when this count becomes zero is the file ready to be deleted from the disk.

```bash

$ touch foo
$ echo hello > foo
$ ln foo foo-hard
$ stat foo
  File: foo
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d      Inode: 1289616 <--***----     Links: 2
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-07-21 10:16:13.405015668 +0000
Modify: 2023-07-21 10:16:23.133033843 +0000
Change: 2023-07-21 10:16:29.941046564 +0000
 Birth: 2023-07-21 10:16:13.405015668 +0000
$
$ stat foo-hard
  File: foo-hard
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d      Inode: 1289616 <--***----     Links: 2
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-07-21 10:16:13.405015668 +0000
Modify: 2023-07-21 10:16:23.133033843 +0000
Change: 2023-07-21 10:16:29.941046564 +0000
 Birth: 2023-07-21 10:16:13.405015668 +0000
$
$
$ ln -s foo foo-soft
$ stat foo-soft
  File: foo-soft -> foo <--***----
  Size: 3               Blocks: 0          IO Block: 4096   symbolic link
Device: 801h/2049d      Inode: 1289887 <--***----     Links: 1
Access: (0777/lrwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-07-21 10:16:57.677098374 +0000
Modify: 2023-07-21 10:16:51.645087108 +0000
Change: 2023-07-21 10:16:51.645087108 +0000
 Birth: 2023-07-21 10:16:51.645087108 +0000
$
$ cat foo
hello
$ cat foo-hard
hello
$ cat foo-soft
hello
```



### Reading and Writing

The following `reads` and `writes` are called when we call `open()` on a file `/foo/bar` and 3 `reads` post that. This is in order to first traverse the inode hierarchy from the root (which is inode no 2 in most systems)

{{< figure
		  src="file_imgs/open_bar.png"
		  caption="From OSTEP Ch 40"
>}}


Below diagram is for creating a new file `/foo/bar`. Notice the huge number of reads and writes just to create a file. Then to write anything, we need to keep updating the free list bitmaps and the actual blocks. We already see how inefficient doing this every single time when something is changed (like editing a file from an editor) is. There are techniques discussed in the next section which solve just this.

{{< figure
		  src="file_imgs/create.png"
		  caption="From OSTEP Ch 40"
>}}


### Caching and Buffering

For reads,
- To remedy what would clearly be a huge performance problem, most file systems aggressively use system memory (DRAM) to cache important blocks.
- Early systems used a static partitioning approach but this is not flexible and wastes resources when not required. Modern systems, in contrast, employ a dynamic partitioning approach. Specifically, many modern operating systems integrate virtual memory pages and file system pages into a unified page cache \[S00\]. In this way, memory can be allocated more flexibly across virtual memory and file system, depending on which needs more memory at a given time.

For writes,
- Clearly performing write every time is a costly operation, so don't 
- Buffer multiple writes and perform them at once, thereby batching them to save multiple list traversals, accesses, and writes
- This has multiple benefits:
	- First, by delaying writes, the file system can batch some updates into a smaller set of I/Os; for example, if an inode bitmap is updated when one file is created and then updated moments later as another file is created, the file system saves an I/O by delaying the write after the first update. 
	- Second, by buffering a number of writes in memory, the system can then schedule the subsequent I/Os and thus increase performance. Finally, some writes are avoided altogether by delaying them; for example, if an application creates a file and then deletes it, delaying the writes to reflect the file creation to disk avoids them entirely. In this case, laziness (in writing blocks to disk) is a virtue


- Some applications (such as databases) don’t enjoy this trade-off. Thus, to avoid unexpected data loss due to write buffering, they simply force writes to disk, by calling fsync(), by using direct I/O interfaces that work around the cache, or by using the raw disk interface and avoiding the file system altogether


## Misc stuff

To see what is mounted on your system, and at which points, simply run the mount program. You’ll see something like this:

```bash
$ mount
/dev/sda1 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
/dev/sda5 on /tmp type ext3 (rw)
/dev/sda7 on /var/vice/cache type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
AFS on /afs type afs (rw)
```

This crazy mix shows that a whole number of different file systems, including ext3 (a standard disk-based file system), the proc file system (a file system for accessing information about current processes), tmpfs (a file system just for temporary files), and AFS (a distributed file system) are all glued together onto this one machine’s file-system tree.


## Closing Notes

1. Well, that's it! Phew! 
2. Review different file systems would be the natural next step for those interested in learning more
3. This article by Dan Luu goes further talking about files. However, beware to first read the STEP chapters before diving there - [Files are hard](https://danluu.com/file-consistency/)

