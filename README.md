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

## 🛠️ Compiling the Driver  
Use the **kernel's kbuild system** via a `Makefile`:  
```makefile  
obj-m += myDev.o  
  
all:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules  
  
clean:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean  
```  
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
> ℹ️ Use `modprobe` if dependencies need to be resolved automatically.  

## ✅ Testing the Driver  
Write and read to/from the device file:  
```bash  
echo "Hello" > /dev/myDev  
cat /dev/myDev  
```  
Or compile and run the test program:  
```bash  
gcc test.c -o test  
sudo ./test HelloWorld  
```  

## ⚠️ Notes  
- You need **root access** (`sudo`) to load/unload kernel modules and access device files.  
- This project uses the **old-style character driver interface**, which is still supported in modern Linux kernels.  
- Ensure the major number is not already in use.  



