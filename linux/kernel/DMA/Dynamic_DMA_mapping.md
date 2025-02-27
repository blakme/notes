# [Dynamic DMA mapping](https://lwn.net/2001/0712/a/dma-interface.php3)

- [Dynamic DMA mapping](#dynamic-dma-mapping)
  - [What memory is DMA'able?](#what-memory-is-dmaable)
  - [DMA addressing limitations](#dma-addressing-limitations)
  - [Types of DMA mappings](#types-of-dma-mappings)
  - [Using Consistent DMA mappings](#using-consistent-dma-mappings)
  - [DMA Direction](#dma-direction)
  - [Using Streaming DMA mappings](#using-streaming-dma-mappings)

Most of the 64bit platforms have special hardware that translates **bus addresses** (DMA addresses) to **physical addresses** similarly to how page tables and/or TLB translate virtual addresses to physical addresses. This is needed so that e.g. PCI devices can access with a Single Address Cycle (32bit DMA address) any page in the 64bit physical address space. Previously in Linux those 64bit platforms had to set artificial limits on the maximum RAM size in the system, so that the `virt_to_bus()` static scheme works (the DMA address translation tables were simply filled on bootup to map each bus address to the physical page `__pa(bus_to_virt())`).

So that Linux can use the **dynamic DMA mapping**, it needs some help from the drivers, namely it has to take into account that DMA addresses should be **mapped only for the time they are actually used and unmapped after the DMA transfer**.

The following API will work of course even on platforms where no such hardware exists, see e.g. `include/asm-i386/pci.h` for how it is implemented on top of the `virt_to_bus` interface.

First of all, you should make sure

    #include <linux/pci.h>

is in your driver. This file will obtain for you the definition of the **`dma_addr_t`** type which should be used everywhere you **hold a DMA (bus) address** returned from the DMA mapping functions.

## What memory is DMA'able?

The first piece of information you must know is what kernel memory can be used with the DMA mapping facilitites. There has been an unwritten set of rules regarding this, and this text is an attempt to finally write them down.

If you acquired your memory via the page allocator (i.e. `__get_free_page*()`) or the generic memory allocators (i.e. `kmalloc()` or `kmem_cache_alloc()`) then you may DMA to/from that memory using the addresses returned from those routines.

This means specifically that you may _not_ use the memory/addresses returned from vmalloc() for DMA. It is possible to DMA to the _underlying_ memory mapped into a `vmalloc()` area, but this requires walking page tables to get the physical addresses, and then translating each of those pages back to a kernel address using something like `__va()`.

This rule also means that you may not use kernel image addresses (ie. items in the kernel's data/text/bss segment, or your driver's) nor may you use kernel stack addresses for DMA. Both of these items might be mapped somewhere entirely different than the rest of physical memory.

What about block I/O and networking buffers? The block I/O and networking subsystems make sure that the buffers they use are valid for you to DMA from/to.

## DMA addressing limitations

Does your device have any DMA addressing limitations? For example, is your **device** only **capable of driving** the low order 24-bits of address on the PCI bus for DMA transfers? If your device can handle any PCI dma address fully, then please skip to the next section, the rest of this section does not concern your device.

For correct operation, you must interrogate the PCI layer in your device probe routine to see if the PCI controller on the machine can properly support the DMA addressing limitation your device has. This query is performed via a call to pci_dma_supported():

    int pci_dma_supported(struct pci_dev *pdev, dma_addr_t device_mask)

Here, `pdev` is a pointer to the PCI device struct of your device, and `device_mask` is a bit mask describing which bits of a PCI address your device supports. It returns non-zero if your card can perform DMA properly on the machine. If it returns zero, your device can not perform DMA properly on this platform, and attempting to do so will result in undefined behavior.

In the failure case, you have two options:

1. Use some non-DMA mode for data transfer, if possible.
2. Ignore this device and do not initialize it.

It is recommended that your driver print a kernel `KERN_WARNING` message when you do one of these two things. In this manner, if a user of your driver reports that performance is bad or that the device is not even detected, you can ask him for the kernel messages to find out exactly why.

So if, for example, you device can only drive the low 24-bits of address during PCI bus mastering you might do something like:

    if (! pci_dma_supported(pdev, 0x00ffffff))
        goto ignore_this_device;

When DMA is possible for a given mask, the PCI layer must be informed of the mask for later allocation operations on the device. This is achieved by setting the `dma_mask` member of the pci_dev structure, like so:

    #define MY_HW_DMA_MASK 0x00ffffff

    if (! pci_dma_supported(pdev, MY_HW_DMA_MASK))
        goto ignore_this_device;

    pdev->dma_mask = MY_HW_DMA_MASK;

A helper function is provided which performs this common code sequence:

    int pci_set_dma_mask(struct pci_dev *pdev, dma_addr_t device_mask)

Unlike `pci_dma_supported()`, this returns `-EIO` when the PCI layer will not be able to DMA with addresses restricted by that mask, and returns 0 when DMA transfers are possible. If the call succeeds, the `dma_mask` will have been updated so that your driver need not worry about it.

There is a case which we are aware of at this time, which is worth mentioning in this documentation. If your device supports multiple functions (for example a sound card provides playback and record functions) and the various different functions have _different_ DMA addressing limitations, you may wish to probe each mask and only provide the functionality which the machine can handle. It is important that the last call to `pci_set_dma_mask()` be for the most specific mask.

Here is pseudo-code showing how this might be done:

    #define PLAYBACK_ADDRESS_BITS    0xffffffff
    #define RECORD_ADDRESS_BITS    0x00ffffff

    struct my_sound_card *card;
    struct pci_dev *pdev;

    ...
    if (pci_set_dma_mask(pdev, PLAYBACK_ADDRESS_BITS)) {
        card->playback_enabled = 1;
    } else {
        card->playback_enabled = 0;
        printk(KERN_WARN "%s: Playback disabled due to DMA limitations.\n",
               card->name);
    }
    if (pci_set_dma_mask(pdev, RECORD_ADDRESS_BITS)) {
        card->record_enabled = 1;
    } else {
        card->record_enabled = 0;
        printk(KERN_WARN "%s: Record disabled due to DMA limitations.\n",
            card->name);
    }

A sound card was used as an example here because this genre of PCI devices seems to be littered with ISA chips given a PCI front end, and thus retaining the 16MB DMA addressing limitations of ISA.

## Types of DMA mappings

There are two types of DMA mappings:

- **Consistent DMA mappings** which are usually mapped at driver initialization, unmapped at the end and for which the hardware should guarantee that the device and the cpu can access the data in parallel and will see updates made by each other without any explicit software flushing.

  Think of "consistent" as "synchronous" or "coherent".

  Good examples of what to use consistent mappings for are:

  - Network card DMA ring descriptors.
  - SCSI adapter mailbox command data structures.
  - Device firmware microcode executed out of main memory.

  The invariant these examples all require is that any cpu store
  to memory is immediately visible to the device, and vice
  versa.  Consistent mappings guarantee this.

- **Streaming DMA mappings** which are usually **mapped for one DMA transfer**, unmapped right after it (unless you use `pci_dma_sync` below) and for which hardware can **optimize for sequential accesses**.

  This of "streaming" as "asynchronous" or "outside the coherency domain".

  Good examples of what to use streaming mappings for are:

  - Networking buffers transmitted/received by a device.
  - Filesystem buffers written/read by a SCSI device.

  The interfaces for using this type of mapping were designed in such a way that an implementation can make whatever performance optimizations the hardware allows. To this end, when using such mappings you must be explicit about what you want to happen.

Neither type of DMA mapping has alignment restrictions that come from PCI, although some devices may have such restrictions.

## Using Consistent DMA mappings

To allocate and map large (`PAGE_SIZE` or so) consistent DMA regions, you should do:

    dma_addr_t dma_handle;

    cpu_addr = pci_alloc_consistent(dev, size, &dma_handle);

where `dev` is a `struct pci_dev *`. You should pass NULL for PCI like buses where devices don't have `struct pci_dev` (like ISA, EISA). This may be called in interrupt context.

This argument is needed because the DMA translations may be bus specific (and often is private to the bus which the device is attached to).

Size is the length of the region you want to allocate.

This routine will **allocate RAM** for that region, so it acts similarly to `__get_free_pages` (but takes size instead of a page order). If your driver needs regions sized **smaller than a page**, you may prefer using the `pci_pool` interface, described below.

**It returns two values: the virtual address which you can use to access it from the CPU and `dma_handle` which you pass to the card**.

The cpu return address and the DMA bus master address are both guaranteed to be **aligned to the smallest `PAGE_SIZE` order** which is greater than or equal to the requested size. This invariant exists (for example) to guarantee that if you allocate a chunk which is smaller than or equal to 64 kilobytes, the extent of the buffer you receive will not cross a 64K boundary.

To unmap and free such a DMA region, you call:

    pci_free_consistent(dev, size, cpu_addr, dma_handle);

where `dev`, `size` are the same as in the above call and `cpu_addr` and `dma_handle` are the values `pci_alloc_consistent` returned to you. This function **may not be called in interrupt context**.

If your driver needs lots of smaller memory regions, you can write custom code to **subdivide pages** returned by `pci_alloc_consistent`, or you can use the `pci_pool` API to do that. A `pci_pool` is like a `kmem_cache`, but it uses `pci_alloc_consistent` not `__get_free_pages`. Also, it understands common hardware constraints for alignment, like queue heads needing to be aligned on N byte boundaries.

Create a `pci_pool` like this:

    struct pci_pool *pool;

    pool = pci_pool_create(name, dev, size, align, alloc, flags);

The "name" is for diagnostics (like a `kmem_cache` name); `dev` and `size` are as above. The device's hardware alignment requirement for this type of data is "align" (a power of two). The flags are `SLAB_` flags as you'd pass to kmem_cache_create. Not all flags are understood, but `SLAB_POISON` may help you find driver bugs. If you call this in a non-sleeping context (f.e. in_interrupt is true or while holding SMP locks), pass SLAB_ATOMIC. If your device has no boundary crossing restrictions, pass 0 for `alloc`; passing 4096 says memory allocated from this pool must not cross 4KByte boundaries (but at that time it
may be better to go for `pci_alloc_consistent` directly instead).

Allocate memory from a pci pool like this:

    cpu_addr = pci_pool_alloc(pool, flags, &dma_handle);

flags are SLAB_KERNEL if blocking is permitted (not in_interrupt nor holding SMP locks), SLAB_ATOMIC otherwise. Like `pci_alloc_consistent`, this returns two values, `cpu_addr` and `dma_handle`.

Free memory that was allocated from a `pci_pool` like this:

    pci_pool_free(pool, cpu_addr, dma_handle);

where `pool` is what you passed to `pci_pool_alloc`, and `cpu_addr` and `dma_handle` are the values `pci_pool_alloc` returned. This function **may be called in interrupt context**.

Destroy a pci_pool by calling:

    pci_pool_destroy(pool);

Make sure you've called `pci_pool_free` for all memory allocated from a pool before you destroy the pool. This function may not be called in interrupt context.

## DMA Direction

The interfaces described in subsequent portions of this document take a DMA direction argument, which is an integer and takes on one of the following values:

    PCI_DMA_BIDIRECTIONAL
    PCI_DMA_TODEVICE
    PCI_DMA_FROMDEVICE
    PCI_DMA_NONE

One should provide the exact DMA direction if you know it.

- `PCI_DMA_TODEVICE` means "from main memory to the PCI device"

- `PCI_DMA_FROMDEVICE` means "from the PCI device to main memory"

You are _strongly_ encouraged to specify this as precisely as you possibly can.

If you absolutely cannot know the direction of the DMA transfer, specify `PCI_DMA_BIDIRECTIONAL`. It means that the DMA can go in either direction. The platform guarantees that you may legally specify this, and that it will work, but this may be at the cost of performance for example.

The value `PCI_DMA_NONE` is to be used for debugging. One can hold this in a data structure before you come to know the precise direction, and this will help catch cases where your direction tracking logic has failed to set things up properly.

Another advantage of specifying this value precisely (outside of potential platform-specific optimizations of such) is for debugging. Some platforms actually have a write permission boolean which DMA mappings can be marked with, much like page protections in a user program can have. Such platforms can and do report errors in the kernel logs when the PCI controller hardware detects violation of the permission setting.

**Only streaming mappings specify a direction**, consistent mappings implicitly have a direction attribute setting of `PCI_DMA_BIDIRECTIONAL`.

The SCSI subsystem provides mechanisms for you to easily obtain the direction to use, in the SCSI command:

    scsi_to_pci_dma_dir(SCSI_DIRECTION)

Where `SCSI_DIRECTION` is obtained from the `sc_data_direction` member of the SCSI command your driver is working on. The mentioned interface above returns a value suitable for passing into the streaming DMA mapping interfaces below.

For Networking drivers, it's a rather simple affair. For transmit packets, map/unmap them with the `PCI_DMA_TODEVICE` direction specifier. For receive packets, just the opposite, map/unmap them with the `PCI_DMA_FROMDEVICE` direction specifier.

## Using Streaming DMA mappings

The streaming DMA mapping routines **can be called from interrupt context**. There are two versions of each map/unmap, one which map/unmap a **single memory region**, one which map/unmap a **scatterlist**.

To map a single region, you do:

    dma_addr_t dma_handle;

    dma_handle = pci_map_single(dev, addr, size, direction);

and to unmap it:

    pci_unmap_single(dev, dma_handle, size, direction);

You should call `pci_unmap_single` when the DMA activity is finished, e.g. **from interrupt which told you the DMA transfer is done**.

Similarly with scatterlists, you map a region gathered from several regions by:

    int i, count = pci_map_sg(dev, sglist, nents, direction);
    struct scatterlist *sg;

    for (i = 0, sg = sglist; i < count; i++, sg++) {
        hw_address[i] = sg_dma_address(sg);
        hw_len[i] = sg_dma_len(sg);
    }

where `nents` is the number of entries in the sglist.

The implementation is free to merge several consecutive `sglist` entries into one (e.g. if DMA mapping is done with PAGE_SIZE granularity, any consecutive sglist entries can be merged into one provided the first one ends and the second one starts on a page boundary - in fact this is a huge advantage for cards which either cannot do scatter-gather or have very limited number of scatter-gather entries) and returns the actual number of `sg` entries it mapped them to.

Then you should loop `count` times (note: this can be less than `nents` times) and use `sg_dma_address()` and `sg_dma_length()` macros where you previously accessed `sg->address` and `sg->length` as shown above.

To unmap a scatterlist, just call:

    pci_unmap_sg(dev, sglist, nents, direction);

Again, make sure DMA activity finished.

PLEASE NOTE:  The `nents` argument to the `pci_unmap_sg` call must be the _same_ one you passed into the `pci_map_sg` call, it should _NOT_ be the 'count' value _returned_ from the `pci_map_sg` call.

Every `pci_map_{single,sg}` call should have its `pci_unmap_{single,sg}` counterpart, because the bus address space is a shared resource (although in some ports the mapping is per each BUS so less devices contend for the same bus address space) and you could render the machine unusable by eating all bus addresses.

If you need to **use the same streaming DMA region multiple times** and touch the data in between the DMA transfers, just map it with `pci_map_{single,sg}`, after each DMA transfer call either:

    pci_dma_sync_single(dev, dma_handle, size, direction);

or:

    pci_dma_sync_sg(dev, sglist, nents, direction);

and after the last DMA transfer **call one of the DMA unmap routines** `pci_unmap_{single,sg}`. If you don't touch the data from the first `pci_map_*` call till `pci_unmap_*`, then you don't have to call the `pci_sync_*`
routines at all.

Here is pseudo code which shows a situation in which you would need to use the `pci_dma_sync_*()` interfaces.

    my_card_setup_receive_buffer(struct my_card *cp, char *buffer, int len)
    {
        dma_addr_t mapping;

        mapping = pci_map_single(cp->pdev, buffer, len, PCI_DMA_FROMDEVICE);

        cp->rx_buf = buffer;
        cp->rx_len = len;
        cp->rx_dma = mapping;

        give_rx_buf_to_card(cp);
    }

    ...

    my_card_interrupt_handler(int irq, void *devid, struct pt_regs *regs)
    {
        struct my_card *cp = devid;

        ...
        if (read_card_status(cp) == RX_BUF_TRANSFERRED) {
            struct my_card_header *hp;

            /* Examine the header to see if we wish
             * to accept the data.  But synchronize
             * the DMA transfer with the CPU first
             * so that we see updated contents.
             */
            pci_dma_sync_single(cp->pdev, cp->rx_dma, cp->rx_len,
                        PCI_DMA_FROMDEVICE);

            /* Now it is safe to examine the buffer. */
            hp = (struct my_card_header *) cp->rx_buf;
            if (header_is_ok(hp)) {
                pci_unmap_single(cp->pdev, cp->rx_dma, cp->rx_len,
                         PCI_DMA_FROMDEVICE);
                pass_to_upper_layers(cp->rx_buf);
                make_and_setup_new_rx_buf(cp);
            } else {
                /* Just give the buffer back to the card. */
                give_rx_buf_to_card(cp);
            }
        }
    }

Drivers converted fully to this interface should not use `virt_to_bus` any longer, nor should they `use bus_to_virt`. Some drivers have to be changed a little bit, because there is no longer an equivalent to `bus_to_virt` in the dynamic DMA mapping scheme - you have to always store the DMA addresses returned by the `pci_alloc_consistent`, `pci_pool_alloc`, and `pci_map_single` calls (`pci_map_sg` stores them in the scatterlist itself if the platform supports dynamic DMA mapping in hardware) in your driver structures and/or in the card registers.
