## Problem with helloworld and the board config fix

The ZYNQ UltraScale+ ZCU102 is a cool FPGA board with a Cortex-A53 and a Cortex-R5. It is my first time to set my hands to an FPGA board. I started with a tutorial, https://www.xilinx.com/support/documentation/sw_manuals/xilinx2019_1/ug1209-embedded-design-tutorial.pdf, which guides through generating hardware layout with Xilinx Vivado (not PL), running a simple bootloader and helloworld program, and then building and running a Linux (on the Cortex hard cores). Our software versions are Ubuntu 18.04.3, Xilinx Vivado 2019.1.2, and Xilinx SDK 2019.1.

Unfortunately, there is no print on the serial terminal, even I follow the tutorial Chapter 2 step by step. My labmate, Seyed Mohammadjavad Seyed Talebi, repeated the tutorial and had the same results. We began to look for solutions online and found these forum questions, https://forums.xilinx.com/t5/Evaluation-Boards/Hello-World-on-Zynq-Ultrascale/td-p/953650, https://forums.xilinx.com/t5/Evaluation-Boards/Hello-World-on-ZCU102/td-p/959547, and these patches https://www.xilinx.com/support/answers/71961.html, https://www.xilinx.com/support/answers/72113.html. As the forum suggests, we tried to first create and run a First Stage Boot Loader, and then run the helloworld application without `psu_init.tcl`, all using the predefined ZCU102_hw_platform rather than the one we built with Vivado. Now the helloworld works.

![helloworld program using predefined hardware platform](./pic2.png)

One problem remains. The hardware platform that we generated with Vivado (Board ZCU102 v3.3) does not work. The forum answers do not seem to solve the problem. I decided to take a look at the hardware platform folder. The folder consists of a couple of giant c and header files and a binary hardware layout file. I attempted to `diff -r` the predefined folder and the one we generated with Vivado, but there are too many differences in the code, including many address differences, which doesn't matter. I did not know how to diff the binary layout file, but I found there is a html summary file showing the hardware setups in a readable format. I diffed the file and found the root cause of the problem! The predefined hardware platform has PMU GPO 0 - PMU GPO 5 enabled, but our build only has PMU GPO 2 enabled. 

![hardware configs html](./pic3.png)

"PMU GPOs are used for sending signals to power supplies and communicating errors," and each of the register banks is reserved for a specific purpose. For example, GPO 0 is "dedicated to the PMU features" (https://www.xilinx.com/support/documentation/user_guides/ug1085-zynq-ultrascale-trm.pdf). Note that GPO 4 and GPO 5 are not documented in the trm pdf. I am not sure why our predefined hardware does not have these GPOs enabled. But trying to enable these registers should gap the difference between our hardware platform and the predefined one.

![config GPOs](./pic4.png)

Now, the bootloader prints and the helloworld prints show up! The root problem seems to be the Platform Management Unit GPOs being mystically disabled.

![helloworld prints](./pic5.png)
