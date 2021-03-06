# Abstract

This directory contains a sample application to demonstrate how mount namespaces work.

# Warning #1

THIS CODE IS INTENDED FOR DEMONSTRATION PURPOSES ONLY AND IS NOT SUITABLE FOR A PRODUCTION ENVIRONMENT!

# Warning #2

YOU SHOULD PROBABLY NOT RUN THIS ON YOUR LAPTOP! This codebase messes with mount points, etc and could destroy your
files. Or eat your cat. And maybe burn down your house. You have been warned.

## Recommended reading

- [Under the hood of Docker](https://pasztor.at/blog/under-the-hood-of-docker)
- [Mount namespaces and shared subtrees](https://lwn.net/Articles/689856/)

## Detailed modus operandi

1. First of all, the program launches a function (`child`) in a separate namespace:
   ```c
   pid_t childPid = clone(
       child,
       stackTop,
       CLONE_NEWNS | SIGCHLD,
       {}
   );
   ```
2. The child is launched in a separate process, which is now in a new namespace. Mount namespaces
   are implicit, so no need to declare a namespace above.
3. However, the mount points are still shared with the parent namespace, so we now read the list of
   mount points from `/proc` using `setmntent`.
4. For every mount point, we remount the mount points as *slaves*:
   ```c
   mount(nullptr, mountEntry->mnt_dir, mountEntry->mnt_type, MS_REC|MS_SLAVE, nullptr)
   ```
   Mounting as a slave means that we can now safely unmount or remount the mount point without
   affecting the parent namespace.
