# Verilog语言学习

## 模块声明类语法：module.........endmodule

```verilog
module xxxxxx_prj(<端口信号列表> ...);
    <逻辑代码>.....
endmodule
```

## 端口声明：input,output,inout

```verilog
input clk
input wire_rst_n
input [7:0] data_in//[最大值：最小值]
```

## 一个基本的module,其中parameter用于声明一些常量

```verilog
module <模块命名>(<端口命名1>,<端口命名2>....);
    //输入端口声明
    input<端口命名1>；
    input wire <端口命名2>；
    input[<最高位>:<最低位>]<端口命名3>；
    ...

    //输出端口声明
    output<端口命名4>;
    output [<最高位>:<最低位>]<端口命名5>;
    output reg[<最高位>:<最低位>]<端口命名6>;
    ...

    //双向（输入/输出）端口声明
    inout<端口命名7>；
    inout[<最高位>:<最低位>]<端口命名8>;
    ...

    //参数定义
    parameter<参数命名1> = <默认值1>;
    parameter[<最高位>:<最低位>]<参数命名2> = <默认值1>;
    ...

    //具体功能逻辑代码
    ...

endmodule
```

## 信号类型：wire（两个寄存器之间的线），reg（寄存器register）等

```verilog
//定义一个wire信号
wire<wire变量名>；

//给一个定义的wire信号直接连接赋值
//该定义等同于分别定义一个wire信号和使用assign语句进行赋值
wire<wire变量名> = <常量或变量赋值>；

//定义一个多位的wire信号
wire[<最高位>:<最低位>]<wire变量名>;

//定义一个reg信号
reg <reg变量名>;

//定义一个赋初值的reg信号
reg<reg变量名> = <初始值>;

//定义一个多位的reg信号
reg[<最高位>:<最低位>]<reg变量名>;

//定义一个赋初值的多位的reg信号
reg[<最高位>:<最低位>]<reg变量名> = <初始值>;

//定义一个二维的多位reg信号
reg[<最高位>:<最低位>]<reg变量名>[<最高位>:<最低位>];
```

## 多语句定义:begin....end(就是C语言里的{}符合，可用于单个语法的多个语句定义)

```verilog
//含有命令的begin语句
begin : <块名>
    //可选声明部分
    //具体逻辑
end

//基本的begin语句
begin
    //可选声明部分
    //具体逻辑
end
```

## 比较判断：if...else，case.....default......endcase

```verilog
if(<判断条件>)
begin
    //具体逻辑
end
//结构与C语言一致，可if...else；if...else if...else ,具体语法参照if的即可


//case语句
case(<判断变量>)
    <取值1>:<具体逻辑1>
    <取值2>:<具体逻辑2>
    default:<具体逻辑2>
endcase
```
