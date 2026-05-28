# Linux Kernel Modules in Embedded Linux

## Overview

Linux Kernel Modules (LKMs) are pieces of code that can be:
- dynamically loaded into the Linux kernel
- unloaded from the kernel at runtime

Kernel modules extend kernel functionality without rebuilding or rebooting the kernel.

Common module types:
- device drivers
- filesystem drivers
- network protocols

---

# 1. What is a Kernel Module?

A kernel module is:

```text
Dynamically Loadable Kernel Code
```

Usually stored as:

```text
.ko file
```

Example:

```text
uart_driver.ko
```

---

# 2. Why Kernel Modules are Needed

Without modules:
- every driver must be built into kernel
- kernel becomes huge
- reboot required for updates

Modules allow:
- dynamic driver loading
- smaller kernels
- easier debugging

---

# 3. Kernel Module Boot Flow

```text
Linux Kernel Boot
        ↓
Root Filesystem Mounted
        ↓
Kernel Module Loaded
        ↓
Driver Registration
        ↓
Hardware Functional
``` id="boot1"

---

# 4. Kernel Architecture with Modules

```text
+--------------------------------+
| User Applications              |
+--------------------------------+
| Linux Kernel                   |
|--------------------------------|
| Core Kernel                    |
| Scheduler                      |
| Memory Manager                 |
| VFS                            |
|--------------------------------|
| Loadable Modules (.ko)         |
|  ├── UART Driver               |
|  ├── WiFi Driver               |
|  └── Filesystem Driver         |
+--------------------------------+
| Hardware                       |
+--------------------------------+
``` id="arch1"

---

# 5. Types of Kernel Modules

| Module Type | Purpose |
|-------------|---------|
| Device Drivers | Hardware support |
| Filesystem Drivers | ext4/FAT support |
| Network Drivers | Ethernet/WiFi |
| Protocol Modules | TCP/IP extensions |

---

# 6. Module File Format

Kernel modules use:

```text
ELF format
```

with extension:
```text
.ko
```

---

# 7. Module Lifecycle

```text
Load Module
     ↓
Initialize Module
     ↓
Register Resources
     ↓
Module Active
     ↓
Unload Module
     ↓
Cleanup Resources
``` id="life1"

---

# 8. Basic Kernel Module Example

```c
#include <linux/module.h>
#include <linux/kernel.h>

static int __init my_init(void)
{
    printk("Module Loaded\n");
    return 0;
}

static void __exit my_exit(void)
{
    printk("Module Removed\n");
}

module_init(my_init);
module_exit(my_exit);

MODULE_LICENSE("GPL");
``` id="mod1"

---

# 9. Important Module Macros

| Macro | Purpose |
|------|---------|
| module_init() | Entry function |
| module_exit() | Exit function |
| MODULE_LICENSE() | License declaration |

---

# 10. __init Macro

```c
__init
```

Marks:
```text
initialization-only code
```

Memory freed after init completes.

---

# 11. __exit Macro

```c
__exit
```

Marks:
```text
module cleanup code
```

---

# 12. Kernel Logging

Modules use:

```c
printk()
```

for logging.

---

# 13. printk Example

```c
printk(KERN_INFO "Driver Loaded\n");
``` id="print1"

---

# 14. Kernel Log Levels

| Level | Meaning |
|------|---------|
| KERN_INFO | Informational |
| KERN_WARNING | Warning |
| KERN_ERR | Error |

---

# 15. Module Compilation

Modules compiled using:
```text
Kbuild system
```

---

# 16. Example Makefile

```make
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
``` id="mk1"

---

# 17. Build Module

```bash
make
``` id="build1"

---

# 18. Generated Files

| File | Purpose |
|------|---------|
| hello.ko | Kernel module |
| hello.o | Object file |
| Module.symvers | Symbol info |

---

# 19. Loading Modules

Use:
```bash
insmod
```

---

## Example

```bash
sudo insmod hello.ko
``` id="ins1"

---

# 20. Removing Modules

Use:
```bash
rmmod
```

---

## Example

```bash
sudo rmmod hello
``` id="rm1"

---

# 21. modprobe Command

Preferred module loader.

Automatically loads dependencies.

Example:

```bash
modprobe usb_storage
``` id="modprobe1"

---

# 22. Listing Loaded Modules

```bash
lsmod
``` id="ls1"

---

# 23. Checking Kernel Logs

```bash
dmesg
``` id="dmesg1"

---

# 24. Module Loading Flow

```text
insmod/modprobe
      ↓
Kernel Loads .ko
      ↓
Resolve Symbols
      ↓
Execute module_init()
      ↓
Driver Active
``` id="load1"

---

# 25. Symbol Resolution

Modules can use:
```text
kernel exported symbols
```

Examples:
- printk
- kmalloc
- request_irq

---

# 26. Exporting Symbols

Kernel functions exported using:

```c
EXPORT_SYMBOL(function_name);
``` id="exp1"

---

# 27. Module Parameters

Modules can accept parameters.

---

## Example

```c
static int value = 10;
module_param(value, int, 0);
``` id="param1"

---

# 28. Loading with Parameters

```bash
insmod hello.ko value=20
``` id="param2"

---

# 29. Module Dependencies

Some modules require:
- other modules
- kernel subsystems

Handled by:
```text
modprobe
```

---

# 30. Device Driver Modules

Most common module type.

Examples:
- UART driver
- WiFi driver
- GPIO driver

---

# 31. Character Driver Module Flow

```text
Load Module
    ↓
Register Char Device
    ↓
Create /dev Node
    ↓
User App Access
``` id="char1"

---

# 32. Filesystem Modules

Examples:
- ext4.ko
- vfat.ko

Loaded dynamically when filesystem mounted.

---

# 33. Network Modules

Examples:
- Ethernet drivers
- WiFi drivers
- protocol extensions

---

# 34. Kernel Module Memory

Modules loaded into:
```text
kernel space
```

---

# 35. Kernel Space Risks

Bad modules can:
- crash kernel
- corrupt memory
- cause kernel panic

---

# 36. Module Initialization Sequence

```text
Kernel Loads Module
      ↓
Allocate Memory
      ↓
Resolve Symbols
      ↓
Call module_init()
``` id="init1"

---

# 37. Module Cleanup Sequence

```text
rmmod
   ↓
Call module_exit()
   ↓
Free Resources
   ↓
Unload Module
``` id="exit1"

---

# 38. Kernel Module and Device Tree

Drivers often use:
```text
Device Tree compatible strings
```

for matching.

---

# 39. Device Tree Match Example

```c
.compatible = "vendor,mydevice"
```

matches:

```dts
compatible = "vendor,mydevice";
``` id="dt1"

---

# 40. Common Kernel APIs Used

| API | Purpose |
|-----|---------|
| kmalloc | Memory allocation |
| kfree | Free memory |
| printk | Logging |
| request_irq | Register interrupt |
| copy_to_user | Send data to userspace |

---

# 41. Interrupt Handling in Modules

Example flow:

```text
Hardware IRQ
      ↓
ISR in Module
      ↓
Handle Device Event
``` id="irq1"

---

# 42. Auto-Loading Modules

udev/modprobe can auto-load:
- USB drivers
- filesystem drivers
- network drivers

---

# 43. Module Information

View module details:

```bash
modinfo hello.ko
``` id="info1"

---

# 44. Example modinfo Output

```text
license: GPL
author: Developer
description: Test Module
```

---

# 45. Kernel Module Source Tree

```text
driver/
 ├── Makefile
 ├── hello.c
 └── Kconfig
```

---

# 46. Embedded Linux and Modules

Embedded systems may:
- use static kernel
- use modular kernel

depending on:
- memory constraints
- flexibility needs

---

# 47. Advantages of Kernel Modules

| Advantage | Description |
|-----------|-------------|
| Dynamic loading | No reboot |
| Smaller kernel | Reduced size |
| Easy updates | Replace modules |
| Easier debugging | Independent testing |

---

# 48. Disadvantages

| Issue | Description |
|------|-------------|
| Runtime overhead | Dynamic linking |
| Kernel crashes | Unsafe code |
| Dependency issues | Missing symbols |

---

# 49. Complete Kernel Module Workflow

```text
Write Module Code
       ↓
Compile Module
       ↓
Generate .ko
       ↓
Load with insmod/modprobe
       ↓
Kernel Executes module_init()
       ↓
Module Active
       ↓
Remove with rmmod
``` id="final1"

---

# 50. Summary

- Kernel modules are dynamically loadable kernel components
- Stored as `.ko` files
- Extend kernel functionality without rebuilding kernel
- Commonly used for device drivers
- Loaded using `insmod` or `modprobe`
- Essential in embedded Linux systems

---

````
