开启kaslr无法下断点问题：
  需要重新加载vmlinux的地址，如果有驱动，驱动的地址也要加载
  vmlinx的加载地址 cat /proc/kallsyms | grep -E " _text$"   // https://blog.csdn.net/gatieme/article/details/104266966
  驱动的地址：cat /sys/module/core/sections/.text   //此处为core驱动
  统一用 add-symbol-file ./vmlinux 地址
