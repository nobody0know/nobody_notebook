# 嵌入式Linux 驱动开发通用知识

## 设备树的标签和别名

如下代码部分，`uart1: serial@80000000` 这就是一个标签，标签就是用如uart1来标识serial@80000000这一长串名字，提升可读性。而别名的作用就是快速找到设备树节点，如在友善的设备树中就对各外设有定义别名

```c
/dts-v1/;

/ {
    // 别名定义
    aliases {
        uart1 = &uart1; // uart1 别名指向名为 uart1 的设备节点
        uart2 = &uart2; // uart2 别名指向名为 uart2 的设备节点
        uart3 = "/serial@10000000"; // uart3 别名指向路径为 /serial@10000000 的设备节点,这里的/表示根目录
    };
    // 串口设备示例，地址不同，uart1是标签
    uart1: serial@80000000 { 
        node_add1{
        };
    };
    // I2C 控制器及设备示例，i2c1是标签
    i2c1: i2c@91000000 {
    };
    // 串口设备示例，地址不同，uart3 是别名，通过路径方式定义
    serial@10000000 {
        // 可在此处添加串口设备的配置信息
    }; 
};

&uart1{ // 通过引用标签的方式往 serial@80000000 中追加一个节点非覆盖。
    node_add2{
    };
};
```

```c
__symbols__ {
        osc24M = "/clocks/osc24M_clk";
        osc32k = "/clocks/osc32k_clk";
        iosc = "/clocks/internal-osc-clk";
        de = "/display-engine";
        display_clocks = "/soc/clock@1000000";
        hdmi = "/soc/hdmi@1ee0000";
```

## 设备树属性

属性是键值对，定义了节点的硬件相关参数，属性有很多种我们下面只讲常用的标准属性，其他属性大家用到的时候再查。属性有对应的值，其中值的类型也有好几种，各种属性我们等会一一列举，我们先把属性能填哪些值搞明白。在设备树中，属性的值类型可以有多种，这些类型通常用于描述设备或子系统的各种特性和配置需求。以下是一些常见的属性值类型：

1. **字符串（String）**:
   
   1. 属性名称：`compatible`
   
   2. 示例值：`compatible = "lckfb,tspi-v10", "rockchip,rk3566";`
   
   3. 描述：指定该设备或节点与哪些设备或驱动兼容。

2. **整数（Integer）**:
   
   1. 属性名称：`reg`
   
   2. 示例值：`reg = <0x1000>;`。
   
   3. 描述：定义设备的物理地址和大小，通常用于描述内存映射的I/O资源。

3. **数组（Array）**:
   
   1. 属性名称：`reg`
   
   2. 示例值：`reg = <0x1000,0x10>;`。
   
   3. 描述：定义设备的物理地址和大小，通常用于描述内存映射的I/O资源。

4. **列表（List）**:
   
   1. 属性名称：`interrupts`
   
   2. 示例值：`interrupts = <0 39 4>, <0 41 4>,<0 40 4>;`。
   
   3. 描述：用于定义例如中断列表，其中每个元组可以表示不同的中断属性（如编号和触发类型）。

5. **空值（Empty）**:
   
   1. 属性名称：`regulator-always-on;`
   
   2. 示例值：`regulator-always-on;`
   
   3. 描述：表示该节点下的regulator是永久开启的，不需要动态控制。

6. **引用（Reference）**:
   
   1. 属性名称：`gpios`
   
   2. 示例值：`gpios = <&gpio1 RK_PB0 GPIO_ACTIVE_LOW>;`
   
   3. 描述：提供一个句柄（通常是一个节点的路径或标识符），用于在其他节点中引用该节点。

### **model属性（字符串）**

model的值是字符串，主要是用于描述开发板型号，有助于用户和开发人员识别硬件。一般在最开头能见到，不是什么重要的属性，如夸克开发板中稚晖君就这样定义了他的夸克开发板

```Shell
model = "Pengzhihui Atom-N";
```

### **compatible属性（字符串或字符串列表）**

`compatible`：<mark>这是最最最最最关键的属性之一，它用于标识设备的兼容性字符串</mark>。操作系统使用这个属性来匹配设备与相应的驱动程序。

```shell
#夸克开发板
compatible = "friendlyelec,nanopi-neo-core", "allwinner,sun8i-h3";
```

```shell
#泰山派
rk_headset: rk-headset {    
    compatible = "rockchip_headset"，"rockchip_headset2";
};
```

耳机检测驱动中会通过`"rockchip_headset"`来匹配驱动

`kernel/drivers/headset_observe/rockchip_headset_core.c`

```Shell
.......
static const struct of_device_id rockchip_headset_of_match[] = {    
    { .compatible = "rockchip_headset", },  // 定义设备树匹配项，指定兼容性字符串，与上面的设备树匹配
    {},                                     // 结束符号
};
MODULE_DEVICE_TABLE(of, rockchip_headset_of_match);  // 定义设备树匹配表供内核使用

static struct platform_driver rockchip_headset_driver = {    
    .probe  = rockchip_headset_probe,   // 注册设备探测函数    
    .remove = rockchip_headset_remove,  // 注册设备移除函数    
    .resume = rockchip_headset_resume,  // 注册设备恢复函数    
    .suspend =  rockchip_headset_suspend, // 注册设备挂起函数    
    .driver = {        
        .name   = "rockchip_headset",   // 设备名称        
        .owner  = THIS_MODULE,          // 持有模块的指针        
        .of_match_table = of_match_ptr(rockchip_headset_of_match),  // 设备树匹配表指针    
    },
};
.........
```

### **reg属性（地址，长度对）**

描述了设备的物理地址范围，包括基址与大小，与`address-cells`和`size-cells`结合使用。

```Shell
gmac1: ethernet@fe010000 {    
    reg = <0x0 0xfe010000 0x0 0x10000>;
}

mpu6050@68 {
                compatible = "fire,i2c_mpu6050";
                reg = <0x68>;
                status = "okay";
			};
```

#### #**address-cells属性（整数）和**#**size-cells属性（整数）**

用于说明父节点如何解释它的子节点中的`reg`属性。

`reg` 属性的一般格式：

```Shell
reg = <[address1] [length1] [address2] [length2] ...>;
```

- `[addressN]`：表示区域的起始物理地址。用多少个无符号整数来表示这个地址取决于父节点定义的`#address-cells`的值。例如，如果`#address-cells`为1，则使用一个32位整数表示地址；如果`#address-cells`为2，则使用两个32位整数表示一个64位地址。

- `[lengthN]`：表示区域的长度（或大小）。用多少个无符号整数来表示这个长度同样取决于父节点定义的`#size-cells`的值。

根据`#address-cells`和`#size-cells`的定义，单个[address, length]对可能会占用2、3、4个或更多的元素。

例如，如果一个设备的寄存器空间位于地址`0x03F02000`上，并且占用`0x1000`字节的大小，假设其父节点定义了`#address-cells = <1>` 和 `#size-cells = <1>`，`reg` 属性的表达方式如下：

```Plaintext
reg = <0x03F02000 0x1000>;
```

如果地址是64位的，父节点`#address-cells = <2>` 和 `#size-cells = <1>`，那么`reg`属性可能会这样写，以表示地址`0x00000001 0x03F02000`和大小`0x1000`:

```Plaintext
reg = <0x00000001 0x03F02000 0x1000>;
```

案例

```Shell
/ {
    #address-cells = <2>;
    #size-cells = <2>;
    cpus {
        #address-cells = <2>;
        #size-cells = <0>;
        cpu0: cpu@0 {
            受cpus节点的影响
            #address-cells = <2>;
            #size-cells = <0>;
            所以地址就是0x0，大小就是 0x0
            */
            reg = <0x0 0x0>; 
        };  
    };
    gmac1: ethernet@fe010000 {
        /*
        受根节点的影响
        #address-cells = <2>;
        #size-cells = <2>;
        所以地址就是0xfe010000 ，大小就是 0x10000
        */
        reg = <0x0 0xfe010000 0x0 0x10000>; 
    };
};
```

### **status属性（字符串）**

这个属性非常重要，我们设备树中其实修改的最大的就是打开某个节点或者关闭某个节点，status属性的值是字符串类型的，他可以有以下几个值，最常用的是okay和disabled。

`status` 属性值包括：

- `"okay"`：表示设备是可操作的，即设备当前处于正常状态，可以被系统正常使用。

- `"disabled"`：表示设备当前是不可操作的，但在未来可能变得可操作。这通常用于表示某些设备（如热插拔设备）在插入后暂时不可用，但在驱动程序加载或系统配置更改后可能会变得可用。

- `"fail"`：表示设备不可操作，且设备检测到了一系列错误，且设备不太可能变得可操作。这通常表示设备硬件故障或严重错误。

- `"fail-sss"`：与 `"fail"` 含义相同，但后面的 `sss` 部分提供了检测到的错误内容的详细信息。

```Shell
//用户三色灯
&leds {
    status = "okay";
};
//耳机插入检测，不使用扩展板情况需关闭，否则默认会检测到耳机插入
&rk_headset {
    status = "disabled";
};
```

### **device_type属性（字符串）**

`device_type`属性通常只用于`cpu`节点或`memory`节点。例如，在描述一个CPU节点时，`device_type`可能会被设置为`"cpu"`，而在描述内存节点时，它可能会被设置为`"memory"`。

```Shell
device_type = "cpu";
```

### **自定义属性**

自定义属性需要注意不要和标准属性冲突，而且尽量做到见名知意

```Shell
/ {
   my_custom_node { /* 自定节点 */
       compatible = "myvendor,my-custom-device"; /* 兼容性属性 */
       my_custom_property = <1>; /* 自定义属性，假设为整数类型 */
       my_custom_string_property = "My custom value"; /* 自定义字符串属性 */
   };
};
```

## 内核字符模块框架

```c
#define DEV_NAME "EmbedCharDev"
#define DEV_CNT (1)
#define BUFF_SIZE 128
//定义字符设备的设备号
static dev_t xxx_devno;
//定义字符设备结构体xxx_chr_dev，名字其实无所谓，只是后缀方便好看
static struct cdev xxx_chr_dev;
static int __init chrdev_init(void)
{
   int ret = 0;
   printk("chrdev init\n");
   //第一步
   //采用动态分配的方式，获取设备编号，次设备号为0，
   //设备名称为EmbedCharDev，可通过命令cat /proc/devices查看
   //DEV_CNT为1，当前只申请一个设备编号
   ret = alloc_chrdev_region(&xxx_devno, 0, DEV_CNT, DEV_NAME);
   if (ret < 0) {
   printk("fail to alloc xxx_devno\n");
   goto alloc_err;
 }
 //第二步
 //关联字符设备结构体cdev与文件操作结构体file_operations
 cdev_init(&xxx_chr_dev, &xxx_chr_dev_fops);
 //第三步
 //添加设备至cdev_map散列表中
 ret = cdev_add(&xxx_chr_dev, xxx_devno, DEV_CNT);
 if (ret < 0) {
   printk("fail to add cdev\n");
   goto add_err;
 }
 return 0;

 add_err:
 //添加设备失败时，需要注销设备号
 unregister_chrdev_region(xxx_devno, DEV_CNT);
 alloc_err:
 return ret;
 }

 static void __exit chrdev_exit(void)
{
   printk("chrdev exit\n");
   unregister_chrdev_region(devno, DEV_CNT);
   cdev_del(&chr_dev);
}

#define BUFF_SIZE 128
//数据缓冲区
static char vbuf[BUFF_SIZE];
static struct file_operations chr_dev_fops = {
   .owner = THIS_MODULE,
   .open = chr_dev_open,//应用程序open函数执行时调用的函数指针，具体调用那个看自己函数实现，名字也随便改
   .release = chr_dev_release,
   .write = chr_dev_write,//应用程序write函数执行时调用的函数指针
   .read = chr_dev_read,
 };

//函数使用，注册设备
module_init(chrdev_init);
module_exit(chrdev_exit);
```
