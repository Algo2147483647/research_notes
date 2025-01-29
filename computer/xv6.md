# **xv6**
* xv6 is a modern reimplementation of Sixth Edition Unix in ANSI C for multiprocessor x86 and RISC-V systems. It was created for pedagogical purposes in MIT's Operating System Engineering course in 2006.

# Startup

* Power On
  * processor enters real mode with only 20 bit address bus
    ```
    address = CS << 4 + IP
    ```
  * init register CS and IP
    ```
      CS = 0xf000
      IP = 0xfff0
    ```
    ```
    => address = 0xffff0
    0xffff0-0xfffff (16B): JMP f000:e05b
    ```
    to jump backwards to the body of the BIOS.

* BIOS, Basic Input Output System  
  * BIOS in a PC is "hard-wired" to the physical address range 0x000f0000-0x000fffff, this design ensures that the BIOS always gets control of the machine first after power-up or any system restart - which is crucial because on power-up there is no other software anywhere in the machine's RAM that the processor could execute.

  * Purpose
    * 自检，然后对一些硬件设备做简单的初始化
    * 构建中断向量表加载中断服务程序
    * 将硬盘(通常引导设备就是硬盘)最开始那个扇区 MBR 加载到 0x7c00 ，然后开始执行

* MBR, Master Boot Record
  * Purpose  
    - The MBR is the first sector that must be read when accessing the hard disk after the computer is turned on.
    - The MBR holds the information on how the disc's sectors are divided into partitions.
    - The MBR also contains executable code to function as a loader for the installed Bootloader and operating system.
  * Structure (512B)
    - 引导程序和参数 (446B)
    - 分区表 DPT (64B)
    - End Sign(2B): 0x55aa
  * xv6 does not actually construct MBR.

* BootLoader
  * Purpose
    - Load the operating system into memory.
    - Enter Protected Mode.
    - Build and Load the page table.
    - ... ...
  * src: <bootasm.S> $\to$ <bootmain.c> $\to$ <entry.S>  
  * <bootasm.S>
    * open A20
      ```
      # Physical address line A20 is tied to zero so that the first PCs 
      # with 2 MB would run software that assumed 1 MB.  Undo that.
      seta20.1:
        inb     $0x64,%al               # Wait for not busy
        testb   $0x2,%al
        jnz     seta20.1

        movb    $0xd1,%al               # 0xd1 -> port 0x64, Indicates that preparing to write command to 0x60 port
        outb    %al,$0x64

      seta20.2:
        inb     $0x64,%al               # Wait for not busy
        testb   $0x2,%al
        jnz     seta20.2

        movb    $0xdf,%al               # 0xdf -> port 0x60, open A20
        outb    %al,$0x60
      ```
      After opening A20, 32 address buses can be used with $2^{32} = 4G$ addressing range.

      * What is A20? 
        - The older x86 micro-processors had address bus of 20bits which would total and give access up to 1 MB of memory. The Intel 386 and above had address bus up to 32 bits allowing 4 GB of memory.
        - To keep in compatible with the older processors the Intel introduced a logical OR gate at the 20 bit of the address bus which could be enabled a or disabled, and the A20 is disabled at the startup.
        - It is connected through a P21 line of the keyboard controller which made it possible for the keyboard controller to enable or disable the A20 Gate.

    * Build GDT (Global Descriptor Table)
      ```
      lgdt    gdtdesc      # load gdt
      ```
      ```
      # Bootstrap GDT
      .p2align 2                                # force 4 byte alignment
      gdt:
        SEG_NULLASM                             # null seg
        SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)   # code seg
        SEG_ASM(STA_W, 0x0, 0xffffffff)         # data seg

      gdtdesc:                                  # Load the address and boundary of GDT into GDTR register
        .word   (gdtdesc - gdt - 1)             # sizeof(gdt) - 1 = boundary
        .long   gdt                             # address gdt
      ```
      ```
      #define SEG_NULLASM                                             \
              .word 0, 0;                                             \
              .byte 0, 0, 0, 0

      #define SEG_ASM(type,base,lim)                                  \
              .word (((lim) >> 12) & 0xffff), ((base) & 0xffff);      \
              .byte (((base) >> 16) & 0xff), (0x90 | (type)),         \
                      (0xC0 | (((lim) >> 28) & 0xf)), (((base) >> 24) & 0xff)
      ```
    * set CR0  
      set PE of register CR0 to turn on the Protected, and the number of bits for CPU is changed from 16 bits to 32 bits.
      ```
      movl    %cr0, %eax
      orl     $CR0_PE, %eax
      movl    %eax, %cr0
      ``` 
      jump to ```start32```, 使用长跳刷新流水线，因为目前流水线里有16 bits 实模式下的指令，而后面应该用32 bits 保护模式下的指令.
      ```
      ljmp    $(SEG_KCODE<<3), $start32

      .code32  # Tell assembler to generate 32-bit code now.
      start32:
        # Set up the protected-mode data segment registers
        movw    $(SEG_KDATA<<3), %ax    # Our data segment selector
        movw    %ax, %ds                # -> DS: Data Segment
        movw    %ax, %es                # -> ES: Extra Segment
        movw    %ax, %ss                # -> SS: Stack Segment
        movw    $0, %ax                 # Zero segments not ready for use
        movw    %ax, %fs                # -> FS
        movw    %ax, %gs                # -> GS
      ```

    * jump to <bootmain.c>  
      Set stack top to 0x7c00, Call ```bootmain```
      ```
      # Set up the stack pointer and call into C.
      movl    $start, %esp              # Set stack top to 0x7c00
      call    bootmain
      ```

  * <bootmain.c>  
    Load the kernel into memory and Call entry.  
    The kernel is on disk.
    ```
    void bootmain() {
      struct elfhdr *elf;
      struct proghdr *ph, *eph;
      void (*entry)(void);
      uchar* pa;

      elf = (struct elfhdr*)0x10000;  // scratch space

      // Read 1st page off disk
      readseg((uchar*)elf, 4096, 0);

      // Is this an ELF executable?
      if(elf->magic != ELF_MAGIC)
        return;  // let bootasm.S handle error

      // Load each program segment (ignores ph flags).
      ph = (struct proghdr*)((uchar*)elf + elf->phoff);
      eph = ph + elf->phnum;
      for(; ph < eph; ph++){
        pa = (uchar*)ph->paddr;
        readseg(pa, ph->filesz, ph->off);
        if(ph->memsz > ph->filesz)
          stosb(pa + ph->filesz, 0, ph->memsz - ph->filesz);
      }

      // Call the entry point from the ELF header.
      // Does not return!
      entry = (void(*)(void))(elf->entry);
      entry();
    }
    ```
    |function|means|
    |---|---|
    |void waitdisk()|Wait for the disk to be free and ready|
    |void readsect(void *dst, uint offset)|Read single sector *offset* to *dst*|
    |void readseg(unchar *pa, uint count, uint offset)|read *count* bytes from the sector where *offset* (add 1) to *pa*. Add 1 because the kernel starts from sector 1|
    |||

  * <entry.S>  
    Build page table -> Load page table -> Set CR3 register -> Jump to ```main()```
    ```
    # By convention, the _start symbol specifies the ELF entry point.
    # Since we haven't set up virtual memory yet, our entry point is
    # the physical address of 'entry'.
    .globl _start
    _start = V2P_WO(entry)

    # Entering xv6 on boot processor, with paging off.
    .globl entry
    entry:
      # Turn on page size extension for 4Mbyte pages
      movl    %cr4, %eax
      orl     $(CR4_PSE), %eax
      movl    %eax, %cr4
      # Set page directory
      movl    $(V2P_WO(entrypgdir)), %eax
      movl    %eax, %cr3
      # Turn on paging.
      movl    %cr0, %eax
      orl     $(CR0_PG|CR0_WP), %eax
      movl    %eax, %cr0

      # Set up the stack pointer to the top of the allocated page space.
      movl $(stack + KSTACKSIZE), %esp

      # Jump to main(), and switch to executing at
      # high addresses. The indirect call is needed because
      # the assembler produces a PC-relative instruction
      # for a direct jump.
      mov $main, %eax
      jmp *%eax

    .comm stack, KSTACKSIZE
    ```
* OS, Operating System
  * <main.c>  
    initialize.
    ```
    // Bootstrap processor starts running C code here.
    // Allocate a real stack and switch to it, first
    // doing some setup required for memory allocator to work.
    int main() {
      kinit1(end, P2V(4*1024*1024)); // phys page allocator
      kvmalloc();      // kernel page table
      mpinit();        // detect other processors
      lapicinit();     // interrupt controller
      seginit();       // segment descriptors
      picinit();       // disable pic
      ioapicinit();    // another interrupt controller
      consoleinit();   // console hardware
      uartinit();      // serial port
      pinit();         // process table
      tvinit();        // trap vectors
      binit();         // buffer cache
      fileinit();      // file table
      ideinit();       // disk 
      startothers();   // start other processors
      kinit2(P2V(4*1024*1024), P2V(PHYSTOP)); // must come after startothers()
      userinit();      // first user process
      mpmain();        // finish this processor's setup
    }
    ```

    * Multi-CPU Setup  
      多CPU下, 先启动一个 CPU (称作 BSP, BootStrap Processor) 作为基础去启动其他CPU (称作 AP, Application Processor). BSP 由系统硬件或 BIOS 动态选择决定.
      ```
      mpinit();        // detect other processors
      startothers();   // start other processors
      mpmain();        // finish this processor's setup
      ``` 
      
      ```mpinit()```: BSP 从 MP Configuration Table 中获取多CPU的的配置信息  
        计算机里面专门设有数据 MP Configuration Table 来描述，Meanwhile, Structure ```Floating Pointer``` 来指向 MP Configuration Table。
      ```
      void mpinit() {
        uchar *p, *e;
        int ismp;
        struct mp *mp;
        struct mpconf *conf;
        struct mpproc *proc;
        struct mpioapic *ioapic;

        if((conf = mpconfig(&mp)) == 0)
          panic("Expect to run on an SMP");
        ismp = 1;
        lapic = (uint*)conf->lapicaddr;
        for(p=(uchar*)(conf+1), e=(uchar*)conf+conf->length; p<e; ){
          switch(*p){
          case MPPROC:
            proc = (struct mpproc*)p;
            if(ncpu < NCPU) {
              cpus[ncpu].apicid = proc->apicid;  // apicid may differ from ncpu
              ncpu++;
            }
            p += sizeof(struct mpproc);
            continue;
          case MPIOAPIC:
            ioapic = (struct mpioapic*)p;
            ioapicid = ioapic->apicno;
            p += sizeof(struct mpioapic);
            continue;
          case MPBUS:
          case MPIOINTR:
          case MPLINTR:
            p += 8;
            continue;
          default:
            ismp = 0;
            break;
          }
        }
        if(!ismp)
          panic("Didn't find a suitable machine");

        if(mp->imcrp){
          // Bochs doesn't support IMCR, so this doesn't run on Bochs.
          // But it would on real hardware.
          outb(0x22, 0x70);   // Select IMCR
          outb(0x23, inb(0x23) | 1);  // Mask external interrupts.
        }
      }
      ```

      ```startothers()```: BSP 启动 APs，通过发送 INIT-SIPI-SIPI 消息给 APs  
      ```
      // Start the non-boot (AP) processors.
      static void startothers() {
        extern uchar _binary_entryother_start[], _binary_entryother_size[];
        uchar *code;
        struct cpu *c;
        char *stack;

        // Write entry code to unused memory at 0x7000.
        // The linker has placed the image of entryother.S in
        // _binary_entryother_start.
        code = P2V(0x7000);
        memmove(code, _binary_entryother_start, (uint)_binary_entryother_size);

        for(c = cpus; c < cpus+ncpu; c++){
          if(c == mycpu())  // We've started already.
            continue;

          // Tell entryother.S what stack to use, where to enter, and what
          // pgdir to use. We cannot use kpgdir yet, because the AP processor
          // is running in low  memory, so we use entrypgdir for the APs too.
          stack = kalloc();
          *(void**)(code-4) = stack + KSTACKSIZE;
          *(void(**)(void))(code-8) = mpenter;
          *(int**)(code-12) = (void *) V2P(entrypgdir);

          lapicstartap(c->apicid, V2P(code));

          // wait for cpu to finish mpmain()
          while(c->started == 0)
            ;
        }
      }
      ```

      ```mpmain()```: APs 启动，各个 APs CPU要像 BSP 一样建立自己的一些机制，比如保护模式，分页，中断等等.  
      最后就是调用 scheduler() 可以开始调度执行程序了。
      最后 BSP 本身再执行 mpenter 自身完成启动，到此所有的 CPU 都已经完成启动，也就是计算机的启动工作正式完成，各种环境已经建立好，可以执行各种程序，完成各种任务了。

# File System


# Lock

## SpinLock

## SleepLock

# Reference
1. https://pdos.csail.mit.edu/6.828/2017/schedule.html
2. https://en.wikipedia.org/wiki/Xv6
3. https://zhuanlan.zhihu.com/p/394247844
4. http://kernelx.weebly.com/a20-address-line.html

