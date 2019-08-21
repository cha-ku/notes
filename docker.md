# Docker 

## Docker - Taming the Beast Part 1

This is from [nschoe's  blog](https://nschoe.com/articles/2016-05-26-Docker-Taming-the-Beast-Part-1.html)

Each docker container creates a  *user space* that is separated from the host's and other containers' 

Namespaces - All the processes on a machine run in a single namespace, i.e. they are in some way related to PID 1 (init/systemd)

### Namespace Isolation
Processes can have multiple PIDs from Linux 2.6.24 (2008) onward.

On a typical machine, if one wants to run *nginx*, *init* will start *bash* which can start *nginx*.

*init* will have PID 1, *bash* could have PID 3349, *nginx* could have PID 888, *nginx (worker)* could have PID 889 and so on...

Once these are containerized, *interactivity* of the processes is limited to upto the **perceived** PID 1

So if the isolated bash is killed, only isloated nginx and its child processes will be killed.


```
1 init
    |
    |-- 6728 zsh
    |   |
    |   |-- 6729 tail
    |
    |-- 7839 firefox
    |   |
    |   |-- 7840 firefox (tab 1)
    |   |-- 7841 firefox (tab 2)
    |   |-- (...)
    |
____|___________________________________________
|   |                    isolated process tree  |
|   |                                           |
|   |-- 8937 (1) bash                           |
|   |   |                                       |
|   |   |-- 8938 (4539) nginx                   |
|   |       |                                   |
|   |       |-- 8939 (4540) nginx (worker)      |
|   |                                           |
|___|___________________________________________|
    (...)
```

### Resource Isolation
This is only one  of the levels of isolation Docker offers. Resource utilization isolation (Memory/Disk) should also be provided for complete containerization.

***cgroups*** can help with resource utilization. cgroups is a tool to allocate and monitor the resource usage to a group of processes.
> docker stats relies on cgroups

### File System Isolation

The host file system must be hidden from and a uniform, clean FS should be exposed to the Isolated processes


Docker makes use of **Union File System** with **union mounts** for this. The UFS is not a file system in that it offloads the data storage and retrieval to the underlying system FS, but implements *union mount*s. A union mount is a *unified* view of two or more directories at a specified mount point.

```
dir1/           dir2/
|               |
|-- file1.txt   |-- file4.mp3
|-- file2.ods   |-- file5.txt
|-- file3.iso   |-- file2.ods
                |-- file6.jpg
```

A union mount of dir1 and dir2 at mount point /my/mnt/pt would be 

```
/my/mnt/pt
|
|-- file1.txt
|-- file2.ods
|-- file3.iso
|-- file4.mp3
|-- file5.txt
|-- file6.jpg
```

We can see the contents of both files at the mount point, transparently. In case of similar file names, there is a concept of precedence: The directory which has precedence is exposed. If that file gets deleted, the "other" file is now visible. To combat this, UFS add a layer that masks the deleted file.

So UFS 'shadow' the files unrelated to the current docker layer. So each layer has its own /, /home, /var, /etc, /usr etc. This also allows you to write files which do not pollute the host or other containers' environments.
