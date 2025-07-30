# 🧮 Linux Character Device Driver  
This project demonstrates how to implement a basic **Linux character device driver** in C. It explains the importance of device files, their interaction with user programs, and how to register/unregister a driver in the Linux kernel space.  
> 📝 **Note**: This guide helps understand how a device file interfaces between the user space and kernel space through a custom character device driver.  

## 📘 Introduction  
Linux devices fall into two main categories:  
- **Character devices**: Stream-based access (like files); no seeking.  
- **Block devices**: Access data in blocks (like disks).  

In this project, we simulate a **character device** represented by `/dev/myDev`. This file serves as the interface to interact with our custom driver.  

## ⚙️ Major & Minor Numbers  
Each device is identified by:  
- **Major number**: Identifies the driver associated with the device.  
- **Minor number**: Identifies the specific device instance.  

To view major/minor numbers:  
```bash  
ls -l /dev/random  
```  

## 🧠 How It Works  
When the device file is accessed, Linux:  
1. Checks the major number.  
2. Passes control to the registered driver.  
3. Uses `file_operations` structure callbacks for system calls like `open`, `read`, `write`, `release`.  

Example:  
```c  
static struct file_operations fops = {  
    .read = dev_read,  
    .open = dev_open,  
    .write = dev_write,  
    .release = dev_rls,  
};  
```  
Driver registration:  
```c  
register_chrdev(major_number, "myDev", &fops);  
```  

## 📦 Creating the Device File  
Use `mknod` to create a device file:  
```bash  
sudo mknod /dev/myDev c 89 0  
sudo chmod a+rw /dev/myDev  
```  
> 🔸 `'c'` stands for character device  
> 🔸 `89` is an example major number (must be unique)  

## 🧩 Driver Code Overview  
The Linux device driver:  
- Registers a char device with a major number.  
- Handles open, read, write, and release system calls.  
- Resides in **kernel space** and is loaded as a **kernel module (`.ko`)**.  

<pre> <code> <kbd>📌 Markdown version:</kbd> ```markdown ```c #include <linux/module.h> #include <linux/string.h> #include <linux/fs.h> #include <asm/uaccess.h> MODULE_LICENSE("GPL"); MODULE_DESCRIPTION("Device Driver Demo"); MODULE_AUTHOR("K Krishna Kumar"); static char msg[100] = {0}; static short readPos = 0; static int times = 0; static int dev_open(struct inode *, struct file *); static int dev_rls(struct inode *, struct file *); static ssize_t dev_read(struct file *, char *, size_t, loff_t *); static ssize_t dev_write(struct file *, const char *, size_t, loff_t *); static struct file_operations fops = { .read = dev_read, .open = dev_open, .write = dev_write, .release = dev_rls }; int init_module(void) { int t = register_chrdev(89, "myDev", &fops); if (t < 0) printk(KERN_ALERT "Device registration failed..\n"); else printk(KERN_ALERT "Device registered...\n"); return t; } void cleanup_module(void) { unregister_chrdev(89, "myDev"); } static int dev_open(struct inode *inod, struct file *fil) { times++; printk(KERN_ALERT "Device opened %d times\n", times); return 0; } static ssize_t dev_read(struct file *filp, char *buff, size_t len, loff_t *off) { short count = 0; while (len && (msg[readPos] != 0)) { put_user(msg[readPos++], buff++); count++; len--; } return count; } static ssize_t dev_write(struct file *filp, const char *buff, size_t len, loff_t *off) { short ind = len - 1; short count = 0; memset(msg, 0, 100); readPos = 0; while (len > 0) { msg[count++] = buff[ind--]; len--; } return count; } static int dev_rls(struct inode *inod, struct file *fil) { printk(KERN_ALERT "Device closed\n"); return 0; } ``` ``` </code> </pre> 

## 🛠️ Compiling the Driver  
Use the **kernel's kbuild system** via a `Makefile`:  
```makefile  
obj-m += myDev.o  
  
all:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules  
  
clean:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean  
```  

![Makefile Screenshot](http://i.imgur.com/wAYM9fV.png)  

Compile using:  
```bash  
make  
```  

## 📥 Loading & Unloading the Module  
```bash  
sudo insmod myDev.ko      # Load driver  
sudo dmesg | tail         # Check kernel log  
cat /proc/modules         # Check if loaded  
cat /proc/devices         # Confirm device name  
sudo rmmod myDev          # Unload driver  
```  

![Modules Output](http://i.imgur.com/oabfAzy.png)  
![Devices Output](http://i.imgur.com/yIeN5as.png)  

> ℹ️ Use `modprobe` if dependencies need to be resolved automatically.  

## ✅ Testing the Driver  
Write and read to/from the device file:  
```bash  
echo "Hello" > /dev/myDev  
cat /dev/myDev  
```  

![Testing the Driver](http://i.imgur.com/dpdSmHT.png)  

You can also run the provided C test program:  
```bash  
gcc test.c -o test  
sudo ./test HelloWorld  
```  

![Test Program Code](http://i.imgur.com/PncaPdn.png)  
![Test Program Output](http://i.imgur.com/sB83VaF.png)  

## ⚠️ Notes  
- You need **root access** (`sudo`) to load/unload kernel modules and access device files.  
- This project uses the **old-style character driver interface**, which is still supported in modern Linux kernels.  
- Ensure the major number is not already in use.  

