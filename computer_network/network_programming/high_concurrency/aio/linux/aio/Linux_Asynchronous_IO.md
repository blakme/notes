# [Linux Asynchronous I/O](https://oxnz.github.io/2016/10/13/linux-aio/)

- [Linux Asynchronous I/O](#linux-asynchronous-io)
  - [Introduction](#introduction)
  - [I/O Models](#io-models)
  - [AIO System Calls](#aio-system-calls)
    - [ABI Interface](#abi-interface)
    - [AIO Command](#aio-command)
    - [AIO Context](#aio-context)
    - [Example](#example)
    - [System Tuning](#system-tuning)
  - [libaio](#libaio)
    - [Install](#install)
    - [Syscall Wrappers](#syscall-wrappers)
    - [Helper Functions](#helper-functions)
    - [Example](#example-1)
  - [POSIX asynchronous I/O](#posix-asynchronous-io)
    - [Library](#library)
    - [Interfaces](#interfaces)
  - [References](#references)

## Introduction

Asynchronous I/O (AIO) is a method for performing I/O operations so that the process that issued an I/O request is not blocked till the operation is complished. Instead, after an I/O request is submitted, the process continues to execute its code and can later check the status of the submitted request.

There are several means to accomplish asynchronous I/O in Linux:

- kernel syscalls

- user space library implementation and use system calls internally (`libaio`)

- emulated AIO entirely in the user space without any kernel support (`librt` for now, part of `libc`)

## I/O Models

|Mode|Blocking|Non-blocking|
|-|-|-|
Synchronous|read/write|read/write (O_NONBLOCK)
Asynchronous|I/O multiplexing (select/poll/epoll)|AIO
||||

## AIO System Calls

### ABI Interface

AIO system call entry points are located in `fs/aio.c` file in the kernel’s source code. Types and constants exported to the user space reside in `/usr/include/linux/aio_abi.h` header file.

Linux kernel provides only 5 system calls for performing asynchronoes I/O.

    #include <linux/aio_abi.h>
    int io_setup(unsigned nr_events, aio_context_t *ctxp);
    int io_destroy(aio_context_t ctx);
    int io_submit(aio_context_t ctx, long nr, struct iocb **iocbpp);
    int io_cancel(aio_context_t ctx, struct iocb *, struct io_event *result);
    int io_getevents(aio_context_t ctx, long min_nr, long nr,
        struct io_event *events, struct timespec *timeout);

Every I/O request that is submitted to an AIO context is represented by an I/O control block structure - `struct iocb`

`io_submit()` takes AIO context ID, size of the array and the array itself as the arguments. Notice, that array should contain pointers to the iocb structures, not the structures themself.

`io_submit()`'s return code can be one of the following values:

1. ret = (number of iocbs sumbmitted)

   Ideal case, all iocbs were accepted for processing.

2. 0 < ret < (number of iocbs sumbmitted)

   `io_submit()` system call processes iocbs one by one starting from the first entry in the passed array. If submission of some iocb fails, it stops at this point and returns the index of iocb that failed. There is no way to know what is the exact reason of a failure. However, if the very first iocb submission fails, see point 3.

3. ret < 0

   There are two reasons why this could happen:

   - Some error happened even before `io_submit()` started to iterate over iocbs in the array (e.g., AIO context was invalid).

   - The submission of the very first iocb (cbx[0]) failed).

After iocb is submitted we can perform any other actions without waiting for I/O to complete. For every completed I/O request (successfully or unsuccessfully) kernel creates an `io_event` structure. To obtain the list of io_events (and consequently all completed iocbs) `io_getevents()` system call should be used. When calling io_getevents(), one needs to specify:

1. which AIO context to get events from (ctx variable)

2. a buffer where the kernel should load events to (events varaiable)

3. minimal number of events one wants to get.

   If less then this number of iocbs are currently completed, `io_getevents()` will block till enough events appear. See point 5 for more details on how to control blocking time.

4. maximum number of events one wants to get. This usually is the size of the events buffer (second 1 in our program)

5. If not enough events are available, we don’t want to wait forever. One can specify a relative deadline as the last argument. NULL in this case means to wait infinitely. If one wants `io_getevents()` not to block at all then `timespec` `timeout` structure need to be initialzed to zero seconds and zero nanoseconds.

The return code of `io_getevents` can be:

1. ret = (max number of events)

   All events that fit in the user provided buffer were obtained from the kernel. There might be more pending events in the kernel.

2. (min number of events) <= ret <= (max number of events)

   All currently available events were read from the kernel and no blocking happened.

3. 0 < ret < (min number of events)

   All currently available events were read from the kernel and we blocked to wait for the time user has specified.

4. ret = 0

   no events are available XXX:? does blocking happen in this case?..

5. ret < 0

   an error happened

`struct io_event`

    /* read() from /dev/aio returns these structures. */
    struct io_event {
            __u64           data;           /* the data field from the iocb */
            __u64           obj;            /* what iocb this event came from */
            __s64           res;            /* result code for this event */
            __s64           res2;           /* secondary result */
    };

`struct iocb`

    /*
     * we always use a 64bit off_t when communicating
     * with userland.  its up to libraries to do the
     * proper padding and aio_error abstraction
     */
    
    struct iocb {
            /* these are internal to the kernel/libc. */
            __u64   aio_data;       /* data to be returned in event's data */
            __u32   PADDED(aio_key, aio_reserved1);
                                    /* the kernel sets aio_key to the req # */
    
            /* common fields */
            __u16   aio_lio_opcode; /* see IOCB_CMD_ below */
            __s16   aio_reqprio;
            __u32   aio_fildes;
    
            __u64   aio_buf;
            __u64   aio_nbytes;
            __s64   aio_offset;
    
            /* extra parameters */
            __u64   aio_reserved2;  /* TO DO: use this for a (struct sigevent *) */
    
            /* flags for the "struct iocb" */
            __u32   aio_flags;
    
            /*
             * if the IOCB_FLAG_RESFD flag of "aio_flags" is set, this is an
             * eventfd to signal AIO readiness to
             */
            __u32   aio_resfd;
    }; /* 64 bytes */

### AIO Command

    # /usr/include/linux/aio_abi.h
    enum {
            IOCB_CMD_PREAD = 0,
            IOCB_CMD_PWRITE = 1,
            IOCB_CMD_FSYNC = 2,
            IOCB_CMD_FDSYNC = 3,
            /* These two are experimental.
             * IOCB_CMD_PREADX = 4,
             * IOCB_CMD_POLL = 5,
             */
            IOCB_CMD_NOOP = 6,
            IOCB_CMD_PREADV = 7,
            IOCB_CMD_PWRITEV = 8,
    };

- `IOCB_CMD_PREAD`

  positioned read; corresponds to `pread()` system call.

- `IOCB_CMD_PWRITE`

  positioned write; corresponds to `pwrite()` system call.

- `IOCB_CMD_FSYNC`

  sync file’s data and metadata with disk; corresponds to `fsync()` system call.

- `IOCB_CMD_FDSYNC`

  sync file’s data and metadata with disk, but only metadata needed to access modified file data is written; corresponds to `fdatasync()` system call.

- `IOCB_CMD_PREADV`

  vectored positioned read, sometimes called “scattered input”; corresponds to `preadv()` system call.

- `IOCB_CMD_PWRITEV`

  vectored positioned write, sometimes called “gathered output”; corresponds to `pwritev()` system call.

- `IOCB_CMD_NOOP`

  defined in the header file, but is not used anywhere else in the kernel.

The **semantics of other fields** in the iocb structure **depends on the command specified**.

### AIO Context

AIO context is a set of data structures that the kernel supports to perform AIO.

Every process can have multiple AIO contextes and as such one needs an **identificator** for every AIO context in a process.

A pointer to ctx variable is passed to `io_setup()` as a second argument and kernel fills this variable with a **context identifier**. Interestingly, `aio_context_t` is actually just an `unsigned long` defined in the kernel (`linux/aio_abi.h`) like that:

    typedef unsigned long aio_context_t;

The first argument of `io_setup()` function is the **maximum number of requests** that can simultaneously reside in the context.

`syscall()`

`man syscall`

    #define _GNU_SOURCE         /* See feature_test_macros(7) */
    #include <unistd.h>
    #include <sys/syscall.h>   /* For SYS_xxx definitions */
    
    int syscall(int number, ...);

> syscall() is a small library function that invokes the system call whose assembly language interface has the specified number with the specified arguments. Employing syscall() is useful, for example, when invoking a system call that has no wrapper function in the C library.
>
> syscall() saves CPU registers before making the system call, restores the registers upon return from the system call, and stores any error code returned by the system call in errno(3) if an error occurs.
>
> Symbolic constants for system call numbers can be found in the header file <`sys/syscall.h`>.

### Example

    #include <stdio.h>
    #include <string.h>
    #include <inttypes.h>
    
    #include <unistd.h>
    #include <fcntl.h>
    #include <sys/syscall.h>
    #include <linux/aio_abi.h>
    
    inline int io_setup(unsigned nr, aio_context_t *ctxp) {
        return syscall(__NR_io_setup, nr, ctxp);
    }
    
    inline int io_destroy(aio_context_t ctx) {
        return syscall(__NR_io_destroy, ctx);
    }
    
    inline int io_submit(aio_context_t ctx, long nr, struct iocb **iocbpp) {
        return syscall(__NR_io_submit, ctx, nr, iocbpp);
    }
    
    inline int io_getevents(aio_context_t ctx, long min_nr, long max_nr,
            struct io_event *events, struct timespec *timeout) {
        return syscall(__NR_io_getevents, ctx, min_nr, max_nr, events, timeout);
    }
    
    int main(int argc, char *argv[]) {
        aio_context_t ctx;
        struct iocb cb;
        struct iocb *cbs[1];
        char data[4096];
        struct io_event events[1];
        int ret;
        int fd;
    
        fd = open("/tmp/test", O_RDWR | O_CREAT);
        if (fd < 0) {
            perror("open");
            return -1;
        }
    
        ctx = 0;
    
        ret = io_setup(128, &ctx);
        if (ret < 0) {
            perror("io_setup");
            return -1;
        }
    
        /* setup I/O control block */
        memset(&cb, 0, sizeof(cb));
        cb.aio_fildes = fd;
        cb.aio_lio_opcode = IOCB_CMD_PWRITE;
    
        /* command-specific options */
        int i;
        for (i = 0; i < 4096; ++i)
            data[i] = 'A';
        cb.aio_buf = (uint64_t)data;
        cb.aio_offset = 0;
        cb.aio_nbytes = 4096;
    
        cbs[0] = &cb;
    
        ret = io_submit(ctx, 1, cbs);
        if (ret != 1) {
            if (ret < 0) perror("io_submit");
            else fprintf(stderr, "io_submit failed\n");
            return -1;
        }
    
        /* get reply */
        ret = io_getevents(ctx, 1, 1, events, NULL);
        printf("events: %d\n", ret);
        ret = io_destroy(ctx);
        if (ret < 0) {
            perror("io_destroy");
            return -1;
        }
        return 0;
    }

### System Tuning

    /proc/sys/fs/aio-max-nr
    /proc/sys/fs/aio-nr

## libaio

### Install

    [oxnz@localhost aio]$ sudo yum install libaio-devel
    [oxnz@localhost aio]$ rpm -ql libaio
    /lib64/libaio.so.1
    /lib64/libaio.so.1.0.0
    /lib64/libaio.so.1.0.1
    /usr/share/doc/libaio-0.3.109
    /usr/share/doc/libaio-0.3.109/COPYING
    /usr/share/doc/libaio-0.3.109/TODO
    [oxnz@localhost aio]$ rpm -ql libaio-devel
    /usr/include/libaio.h
    /usr/lib64/libaio.so

### Syscall Wrappers

    /* /usr/include/libaio.h */
    /* Actual syscalls */
    extern int io_setup(int maxevents, io_context_t *ctxp);
    extern int io_destroy(io_context_t ctx);
    extern int io_submit(io_context_t ctx, long nr, struct iocb *ios[]);
    extern int io_cancel(io_context_t ctx, struct iocb *iocb, struct io_event *evt);
    extern int io_getevents(io_context_t ctx_id, long min_nr, long nr, struct io_event *events, struct timespec *timeout);

### Helper Functions

    static inline void io_prep_pread(struct iocb *iocb, int fd, void *buf, size_t count, long long offset)
    static inline void io_prep_pwrite(struct iocb *iocb, int fd, void *buf, size_t count, long long offset)
    static inline void io_prep_preadv(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset)
    static inline void io_prep_pwritev(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset)

    static inline void io_prep_poll(struct iocb *iocb, int fd, int events)
    static inline void io_prep_fsync(struct iocb *iocb, int fd)
    static inline void io_prep_fdsync(struct iocb *iocb, int fd)

    static inline int io_poll(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd, int events)
    static inline int io_fsync(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd)
    static inline int io_fdsync(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd)

    static inline void io_set_eventfd(struct iocb *iocb, int eventfd);

`struct iocb`

    struct io_iocb_poll {
            PADDED(int events, __pad1);
    };      /* result code is the set of result flags or -'ve errno */
    
    struct io_iocb_sockaddr {
            struct sockaddr *addr;
            int             len;
    };      /* result code is the length of the sockaddr, or -'ve errno */
    
    struct io_iocb_common {
            PADDEDptr(void  *buf, __pad1);
            PADDEDul(nbytes, __pad2);
            long long       offset;
            long long       __pad3;
            unsigned        flags;
            unsigned        resfd;
    };      /* result code is the amount read or -'ve errno */
    
    struct io_iocb_vector {
            const struct iovec      *vec;
            int                     nr;
            long long               offset;
    };      /* result code is the amount read or -'ve errno */
    
    struct iocb {
            PADDEDptr(void *data, __pad1);  /* Return in the io completion event */
            PADDED(unsigned key, __pad2);   /* For use in identifying io requests */
    
            short           aio_lio_opcode;
            short           aio_reqprio;
            int             aio_fildes;
    
            union {
                    struct io_iocb_common           c;
                    struct io_iocb_vector           v;
                    struct io_iocb_poll             poll;
                    struct io_iocb_sockaddr saddr;
            } u;
    };
    
    struct io_event {
            PADDEDptr(void *data, __pad1);
            PADDEDptr(struct iocb *obj,  __pad2);
            PADDEDul(res,  __pad3);
            PADDEDul(res2, __pad4);
    };

### Example

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <err.h>
    #include <errno.h>
    
    #include <unistd.h>
    #include <fcntl.h>
    #include <libaio.h>
    
    int main() {
        io_context_t ctx;
        struct iocb iocb;
        struct iocb * iocbs[1];
        struct io_event events[1];
        struct timespec timeout;
        int fd;
    
        fd = open("/tmp/test", O_WRONLY | O_CREAT);
        if (fd < 0) err(1, "open");
    
        memset(&ctx, 0, sizeof(ctx));
        if (io_setup(10, &ctx) != 0) err(1, "io_setup");
    
        const char *msg = "hello";
        io_prep_pwrite(&iocb, fd, (void *)msg, strlen(msg), 0);
        iocb.data = (void *)msg;
    
        iocbs[0] = &iocb;
    
        if (io_submit(ctx, 1, iocbs) != 1) {
            io_destroy(ctx);
            err(1, "io_submit");
        }
    
        while (1) {
            timeout.tv_sec = 0;
            timeout.tv_nsec = 500000000;
            if (io_getevents(ctx, 0, 1, events, &timeout) == 1) {
                close(fd);
                break;
            }
            printf("not done yet\n");
            sleep(1);
        }
        io_destroy(ctx);
    
        return 0;
    }

Compile:

    cc libaio.c -o libaio -laio

## POSIX asynchronous I/O

### Library

    /lib64/librt.so
    /usr/include/aio.h

### Interfaces

The POSIX AIO interface consists of the following functions:

- `aio_read(3)` Enqueue a read request. This is the asynchronous analog of `read(2)`.

- `aio_write(3)` Enqueue a write request. This is the asynchronous analog of `write(2)`.

- `aio_fsync(3)` Enqueue a sync request for the I/O operations on a file descriptor. This is the asynchronous analog of `fsync(2)` and `fdatasync(2)`.

- `aio_error(3)` Obtain the error status of an enqueued I/O request.

- `aio_return(3)` Obtain the return status of a completed I/O request.

- `aio_suspend(3)` Suspend the caller until one or more of a specified set of I/O requests completes.

- `aio_cancel(3)` Attempt to cancel outstanding I/O requests on a specified file descriptor.

- `lio_listio(3)` Enqueue multiple I/O requests using a single function call.

`man 7 aio`

> The current Linux POSIX AIO implementation is provided in user space by glibc. This has a number of limitations, most notably that maintaining multiple threads to perform I/O operations is expensive and scales poorly. Work has been in progress for some time on a kernel state-machine-based implementation of asynchronous I/O (see io_submit(2), io_setup(2), io_cancel(2), io_destroy(2), io_getevents(2)), but this implementation hasn’t yet matured to the point where the POSIX AIO implementation can be completely reimplemented using the kernel system calls.

## References

<http://www.fsl.cs.sunysb.edu/~vass/linux-aio.txt>

[Boost application performance using asynchronous I/O](https://developer.ibm.com/technologies/linux/articles/l-async/)
