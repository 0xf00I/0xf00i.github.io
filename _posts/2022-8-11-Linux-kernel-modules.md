---
title: The linux kernel modules
layout: default
tags: [Programming]
blog_post: true
---

# Introduction 
In this tutorial, I'm going to teach you how to write linux kernel modules, it is necessary to know C programming
language.

You will probably ask - &quot;So, what the hell is that linux kernel module?&quot;
* it is a piece of code that can be dynamically loaded and unloaded from the kernel, &quot;maybe you don't know what kernel is&quot;
***It is the main part of each operating system. It is &quot;program&quot; that is loaded and executed by bootloader at the boot time**  The kernel manages all system resources. It's responsible for communication between software and hardware, manages all user's processes and many, many more.

# Overview  
- Kernel and user mode processes run in different privilege level. New processors support it i.e. Intel processors have the following privilege

```
levels:
ring0(the most powerful privilege level)
ring1
ring2
ring3(the least powerful privilege level)
```

Linux uses only two of them  <code>ring0</code> for kernel and <code>ring3</code> for user mode proceses. You can ask - &quot;What are these privilege levels useful for?&quot;.
If user mode processes run in <code>ring0</code>, they would be able to execute some &quot;destructive&quot; code, For example they could execute &quot;cli&quot; processor command, It would stop all interrupts and as a result stop whole kernel! It would be very bad for safety of the system, That's why only kernel runs in <code>ring0</code> and
user mode processes in <code>ring3</code> when they run in <code>ring3</code> they can't do anything bad to the kernel.

The main power of linux kernel modules is that they run in <code>ring0</code> (kernel mode), not as normal processes in <code>ring3</code>(user mode). Of course only root can load
them :) Why are they useful? For example, there can be a sitation when we have some hardware and unfortunately we haven't drivers for it compiled into the kernel. Then, kernel modules can help us.
Kernel module can be driver for that hardware, We can load such a kernel module and then we can normally use our hardware without kernel module, it would be necessary to recompile the kernel with support for this hardware and it takes really long time...

How to load modules? Modules are usually files with &quot;.ko&quot; extension. All we need to do is to execute command as <code style="color:red">root</code>
insmod module.ko
The module was loaded. But after some time we will want to unload the module, How to do this? We execute again as <code style="color:red">root</code>


```sh
rmmod module
or
rmmod module.ko
# Never mind :)
```

OK. Now we know what are linux kernel modules, why they are useful and how to load them, We can write a simple module then
it will be standard ***Hello world!*** module.

* Read the comments in code the explain many things.


```cpp
/* These are standard module include files */
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

/* This is function that will be executed when we load the module */
int __init mod_init(void)
{
	/* Now we see printk function. This is something like
	standard printf in C. But we are working in kernel mode
	so we can't use functions used by user mode programs.
	I will give more information about this function right
	after the code of this module */
	printk(KERN_ALERT "Hello world!\n");

	return 0;
}

/* This function will be executed when we unload the module */
void __exit mod_exit(void)
{
	printk(KERN_ALERT "Bye world\n");
}


/* Here we register mod_init and mod_exit */

/* mod_init and mod_exit functions can have different names they only have to be registered by module_init and module_exit macros */

/* As a parameter of module_init we give function that has to be executed during loading of the module */
module_init(mod_init);

/* As a parameter of module_exit we give function that has to be executed during unloading of the module */
module_exit(mod_exit);

```


As promised, I will explain printk function, First thing if you want to &quot;normally&quot; see what printk writes you must load and unload module not in "X" but from standard tty console, So  <code>printk</code> just writes given text to the screen if you load module in "X" mode you won't see what printk wrote. However you can still see it
by executing command ```dmesg``` However, <code>printk</code> was not meant to communicate with user it is rather used as a logging mechanism.  ```KERN_ALERT``` is a priority of message to be logged, There are 8 priorities levels each level has its own macro if value of used priority is lower(the lower value it has, the more important is the message)
than console_loglevel the message is written to the screen you can see all macros in ```linux/kernel.h``` file(this is relative path from root kernel's source code directory).

How to compile such a module? We use a special Makefile:


```makefile
obj-m += hello.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=${PWD} modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=${PWD} clean


```

I assume that your file with source code of our module is "hello.c" Now we execute <code>make</code> ok we have "hello.ko" file, Load it as <code style="color:red">root</code> from tty console <code>insmod hello.ko</code> 
You should see "Hello world!" Now you can unload this module <code>rmmod hello</code> You should see "Bye world!" 
You should also include 


```sh
```MODULE_LICENSE``` specifies what license is used for this module.
```MODULE_AUTHOR```  specifies who is he author of this module.
```MODULE_DESCRIPTION``` is a short description what the module does.
```

Let's see modified module:


```cpp
/* These are standard module include files */
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Ormi");
MODULE_DESCRIPTION("Hello World");

/* This is function that will be executed when we load the module */
int __init mod_init(void)
{
	/* Now we see printk function. This is something like
	standard printf in C. But we are working in kernel mode
	so we can't use functions used by user mode programs.
	I will give more information about this function right
	after the code of this module */
	printk(KERN_ALERT "Hello world!\n");

	return 0;
}

/* This function will be executed when we unload the module */
void __exit mod_exit(void)
{
	printk(KERN_ALERT "Bye world\n");
}


/* Here we register mod_init and mod_exit */

/* As a parameter of module_init we give function that has to be executed during loading of the module */
module_init(mod_init);

/* As a parameter of module_exit we give function that has to be executed during unloading of the module */
module_exit(mod_exit);

```


Let's compile it again <code>make</code> Now we can see some information about module, Execute the command <code>modinfo hello.ko</code> We can see something like this:


```makefile
filename:       hello.ko
description:    Hello World
author:         Ormi
license:        GPL
srcversion:     0B4C5D175084D60DBC22242
depends:        
vermagic:       2.6.28-11-generic SMP mod_unload modversions 586 
```

One day we will write a module, which source code will be too big to fit in one file. For example it can fit in two files: one.c - two.c, How to compile it into one module? We can use such a Makefile:


```makefile
obj-m += big.o
big-objs += one.o two.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=${PWD} modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=${PWD} clean

```

When we compile it with <code>make</code> command, we will get "big.ko" file which is our module :) Now you can do some experiments with our modules, try creating something bigger you should be familiar with this because now we are moving
to something more complicated. If you didn't understand what I wrote you can have problems with next things.

## PROCFS

Linux has a nice feature that helps kernel and modules to communicate with processes **procfs** In most of ditributions you can find it in
/proc directory, There are sub-directories for all processes and some other &quot;files&quot;(for example /proc/version which gives us information about kernel's version) In this section I will show how to create &quot;files&quot; or &quot;entries&quot; in **procfs**, I will explain what functions and structures we need for our module.

* ```struct proc_dir_entry``` Each entry in **procfs** is represented by its own ```proc_dir_entry structure``` Let's look at definition of this structure:


```cpp
struct proc_dir_entry {
        unsigned int low_ino;
        unsigned short namelen;
        const char *name;
        mode_t mode;
        nlink_t nlink;
        uid_t uid;
        gid_t gid;
        loff_t size;
        const struct inode_operations *proc_iops;
        /*
         * NULL proc_fops means PDE is going away RSN or
         * PDE is just created. In either case, e.g. read_proc won't be
         * called because it's too late or too early, respectively.
         *
         * If you're allocating proc_fops dynamically, save a pointer
         * somewhere.
         */
        const struct file_operations *proc_fops;
        struct proc_dir_entry *next, *parent, *subdir;
        void *data;
        read_proc_t *read_proc;
        write_proc_t *write_proc;
        atomic_t count;         /* use count */
        int pde_users;  /* number of callers into module in progress */
        spinlock_t pde_unload_lock; /* proc_fops checks and pde_users bumps */
        struct completion *pde_unload_completion;
        struct list_head pde_openers;   /* who did open, but not release */
};

```


We are interested only in following fields:
1. name - name of the entry in profs
2. mode - who can access the entry(for example 777)
3. read_proc - pointer to function that manages reading from this file. 
4. count - how many bytes can we write there.
* Let's look at prototype:

```cpp
typedef int (read_proc_t)(char *page, char **start, off_t off,
                          int count, int *eof, void *data);
```

Here we are interested only in page and count arguments page is pointer to user mode buffer where we have to write data

```cpp
write_proc - pointer to function that manages writing to this file. Prototype:
typedef int (write_proc_t)(struct file *file, const char __user *buffer,
                           unsigned long count, void *data);
```

We are interested only in buffer and count buffer is pointer to user mode buffer in which there is stored data which has to be written to our entry <code>count</code> - size of this buffer
OK. that's all I wanted to say about <code>proc_dir_entry</code>

2. <code>create_proc_entry</code> function

Let's look at prototype:

```cpp
extern struct proc_dir_entry *create_proc_entry(const char *name, mode_t mode, struct proc_dir_entry *parent);
```
* name - name of entry to be created
* mode - this value will be written to mode field of proc_dir_entry structure of create entry
* parent - in which sub-directory of /proc we want to create our entry. We give 0 here, because we want to create our entry right in /proc.

This function return pointer to <code>proc_dir_entry</code> representing out entry in procfs.

What will our module do?
1. It will create entry named &quot;test_proc&quot;.
2. It will have special buffer our_buf where it will store some data.
3. When user writes to our entry(for example by command <code>echo hello &gt; /proc/test_proc</code> data written to this file will be stored in our_buf.
4. When user reads from our entry(for example by command <code>cat /proc/test_proc</code>, module &quot;shows&quot; him the content of our_buf.

Next functions we need:
1. <code>sprintf and snprintf</code>(they are used as standard sprintf and snprintf for user mode programs)
2. <code>copy_from_user(void *dst, void *src, int count)</code>
This function is used to copy data from user mode buffer(src) to our kernel mode buffer(dst). count bytes are copied.
3. <code>remove_proc_entry(char *name, struct proc_dir_entry *parent)</code>
name - name of entry we want to delete
parent - in which sub-directory of /proc is entry we want to delete. We give 0 here, because we want to delete entry in /proc.

Now let's take a look at code of our module:

```cpp
/* Standard includes for modules */
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

/* for proc_dir_entry and create_proc_entry */
#include <linux/proc_fs.h>

/* For sprintf and snprintf */
#include <linux/string.h>

/* For copy_from_user */
#include <linux/uaccess.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Ormi");
MODULE_DESCRIPTION("Simple module using procfs");

static char our_buf[256];

int buf_read(char *buf, char **start, off_t offset, int count, int *eof, void *data)
{
	int len;
	/* For example - when content of our_buf is hello when user executes command cat /proc/test_proc;
	he will see content of our_buf(in our example hello */
	len = snprintf(buf, count, "%s", our_buf);
	return len;
}

/* When user writes to our entry. For example echo aa > /proc/test_ptoc. aa will be stored in our_buf.
Then, when user reads from our entry(cat /proc/test_proc) he will see aa */
static int buf_write(struct file *file, const char *buf, unsigned long count, void *data)
{
	/* If count is bigger than 255, data which user wants to write is too big to fit in our_buf. We don't want
	any buffer overflows, so we read only 255 bytes */	
	if(count > 255)
		count = 255;
	/* Here we read from buf to our_buf */
	copy_from_user(our_buf, buf, count);
	/* we write NULL to end the string */
	our_buf[count] = '\0';
	return count;
}

int __init start_module(void)
{

	/* We create our entry */	
	struct proc_dir_entry *de = create_proc_entry("test_proc", 0666, 0);

	/* Set pointers to our functions reading and writing */
	de->read_proc = buf_read;
	de->write_proc = buf_write;

	/* We initialize our_buf with some text. */
	sprintf(our_buf, "hello");

	return 0 ;
}

void __exit exit_module(void)
{
	/* We delete our entry */
	remove_proc_entry("test_proc", NULL);
}

module_init(start_module);
module_exit(exit_module);

```


Now, edit Makefile and change hello.o to proc.o (if file with source code of this module is written as proc.c) and compile.
<code>make</code> Then load module <code>insmod proc.ko</code> and test our module ;)

## NOTIFIERS

Next thing I want to write about is something called &quot;notify chain&quot;. There are some events, which are quite important, and when they occur some kernel sub-systems want to be informed about this. Here notify chains come to help us.
We will use some keyboard events as an example and then we will write our first, doing-something module :) So:
when user presses a key, kernel &quot;reads&quot; it and then, using notify chain informs all subsystems which want to be informed about pressed key, One of the structures used by notifiers is &quot;notifier_block&quot;
Let's look at the definition:


```cpp
struct notifier_block {
        int (*notifier_call)(struct notifier_block *self, unsigned long x, void *data);
        struct notifier_block *next;
        int priority;
};
```

This structure is used to register in notify chains. ```notifier_call``` is pointer to function which is called when an event occurs. Priority informs about
priority of that function. Functions with higher priorities are called earlier. However, this field is usually set to 0. 
next is pointer to next registered notifier_block, So:
* An event occurs, Kernel &quot;looks&quot; at head of list of notifier_blocks and executes first registered function. Then it goes where pointer next points and executes function, Then it goes again to next and again executes function. "So how can a module register in keyboard notify chain?"
* It creates a notifier_block structure and initializes it. For example:


```cpp
struct notifier_block nb;
nb.notifier_call = function
```

Where function is function which has to be executed when an event occurs(in our example - when a key is pressed).
Registers by register_keyboard_notifer `(struct notifier_block *nb)` . In our example: `register_keyboard_notifier(&amp;nb);`

We have to write function which will handle situation when a key is pressed. What are the parameters? As in the prototype:
(struct notifier_block *self, unsigned long stage, void *data);
self - pointer to our notifier_block - we don't use it
stage - stage of &quot;handling&quot; the pressed key. We do something only when stage is KBD_KEYSYM.
data - pointer to keyboard_notifier_param structure.
In this structure we are interested only in value field. It stores the value of pressed key. 

Our module will be very simple random numbers generator. It's only a toy and can't be treated seriously


```c
/* Standard includes for modules */
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
/* For keyboard_notifier param etc. */
#include <linux/keyboard.h>
/* notifier_block etc. */
#include <linux/notifier.h>
/* create_proc_entry etc. */
#include <linux/proc_fs.h>
/* sprintf */
#include <linux/string.h>

MODULE_LICENSE("GPL") ;

static unsigned long long random_num ;

/* Our function that will be executed when a key is pressed */
static int kbd_notify(struct notifier_block *self, unsigned long stage, void *data)
{
	struct keyboard_notifier_param *param = data; /* Pointer to keyboard_notifier_param */
	int value = param->value - 0xf000; /* Value can't be to big */
	/* We calculate our random number by random_num*key_value
	This is really dummy and improfessional, but this module
	only has to show how to use notifiers */	
	/* Value must fit in long long :) */
	if(random_num > 1000000000)
		random_num -= 10*value;	
	else	
		random_num  *= value;
	return NOTIFY_DONE ;
}

/* We prepare our notifier_block structure. kbd_notify is function handling events */
static struct notifier_block kbd_nb = { .notifier_call = kbd_notify, } ;

/* We register in notify chain */
static void handler_init(void)
{
	register_keyboard_notifier(&kbd_nb) ;
}

/* We write this random number to user mode buffer */
static int random_read(char *buf, char **start, off_t off, int count, int *peof, void *data)
{
	int len = sprintf(buf, "%llu", random_num);

	return len ;
}

/* We create entry in procfs */
static void proc_init(void)
{
	struct proc_dir_entry *de = create_proc_entry("random_simple", 0444, 0);
	de->read_proc = random_read;
}

static int __init random_init(void) 
{
	handler_init();
	proc_init();
	random_num = 1;
	

	return 0 ;
}

static void __exit random_exit(void)
{
	remove_proc_entry("random_simple", 0);
	unregister_keyboard_notifier(&kbd_nb);
}


module_init(random_init) ;
module_exit(random_exit) ;
/*  Now you can compile it and test. */
```

## END
That's all for now. I hope you learned something from this tutorial and became interested in linux kernel. If you want to start browsing kernel's code,
I can give an advise. I began my adventure with linux kernel by writing linux kernel modules one day I got an idea to write a [simple rootkit](https://0x00sec.org/t/writing-a-simple-rootkit-for-linux/29034), I wanted my rootkit to be difficult to detect to achieve this i had to know more organisation of kernel's structures, Firstly, I analysed how <code>procfs</code> works and how new entries are created etc.

### References:
[The Linux Kernel Module Programming Guide by Peter Jay Salzman, Michael Burian and Ori Pomerantz](http://tldp.org/LDP/lkmpg/2.6/html/index.html)
