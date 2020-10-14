# Accessing hardware

This section describes advised methods of accessing hardware registers. Leaving aside architecture dependent methodology, one should access hardware registers by using base address (either from data or I/O address space) and particular register offset:

````C
    /* Device registers */
    enum { reg0 = 0, reg1, reg2, reg3 };

    /* Example base address of device */
    volatile u32 *base = (void *)0x20000000;

    /* Set 19th bit in the register 0 (address 0x20000000 + 0 = 0x20000000) */
    *(base + reg0) |= 1 << 19;

    /* Clear 7th bit int the register 1 (address 0x20000000 + 4 = 0x20000004 */
    *(base + reg1) &= ~(1 << 7);

    /* Read register 2 (address 0x20000000 + 8 = 0x20000008) */
    u32 val = *(base + reg2);
````

## ISA with in/out instructions

Some architectures support in/out instructions and hardware registers are not mapped in the data memory space (e.g. x86). In those architectures it is necessary to use either compiler buildins or an inline assembler. The second option is preffered, as it increases the code portability to another compiler.

For IA32 ISA there are a set of functions defined in <arch/ia32/io.h> which should be used to access hardware registers:


* `u8 inb(void *addr)` - read byte (8 bits) from register located on address addr
* `void outb(void *addr, u8 b)` - write byte (8 bits) b to register located on address addr
* `u16 inw(void *addr)` - read word (16 bits) from register located on address addr
* `void outw(void *addr, u16 w)` - write word (16 bits) b to register located on address addr
* `u32 inl(void *addr)` - read double word (32 bits) from register located on address addr
* `void outl(void *addr, u32 l)` - write double word (32 bits) b to register located on address addr

## ISA without in/out instructions

The following sections describe both the MMU and non MMU approach.

### ISA with MMU

On architectures with MMU (memory management unit) driver developer do not have direct access to the physical memory. To be able to manipulate registers located in the main address space one has to gain access to memory located under particular physical address by mapping this address to the virtual address space. It can be achieved by using mmap syscall:

>
    #include <sys/mman.h>
>
    void *vaddr = mmap(NULL, SIZE_PAGE, PROT_READ | PROT_WRITE, MAP_UNCACHED, OID_PHYSMEM, paddr);

Above example maps one page (typically 4 KiB) of physical memory (as idicated by OID_PHYSMEM) starting from oddress paddr with read (PROT_READ) and write (PROT_WRITE) permissions at the lowest possible virtual address (as idicatated by first argument). Address vaddr returned from mmap should be checked to make sure that it do not contain MAP_FAILED value, which would indicate that mmap failed to perform desired mapping.


### ISA without MMU

On architectures without MMU access to the hardware registers does not require prior memory mapping. Registers can be accessed by directly setting volatile pointer to the desired physical base address.