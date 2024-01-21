# EM Fault Injection on ARM and RISC-V阅读笔记

本篇文章主要介绍了在ARM和RISC-V两种不同架构上的EMFI实现。

在ARM上的汇编代码如下：

``````assembly
_MAIN:
  Assert trigger
  NOPs
  LDR R3, =(0xFEDCBA98);
  LDR R4, =(0xFEDCBA98);
  CMP R3, R4;
  BNE _FAULT;
  Deassert trigger
  B MAIN
``````

基于上述代码进行了三次实验：第一次实验中，BNE 和后续指令LDR R0的OpCode被赋给了R4。R4本应取到PC+0x30处的数据，但是取到了PC+0处的数据，也就是说EMFI导致错误的数据被读入寄存器中。

第二、三次实验则是为了排除EMFI修改了LDR R4的可能性。第二次在LDR R4后加入了多行NOP，第三次实验则是将第二次实验中LDRR4后的第一个NOP改成了MOVS。论文结果表明，是PC没有正确偏移，PC+0xAMN变成了PC+0xA1N。私下认为，这种说法并不妥当，因为当时PC所处的Address恰好是0xA1X，因此跳转到0xA1N有可能是因为此时PC的Address造成的，而且文章并没有证明是否与X值有关。应该说是在PC=0xAB8时偏移计算时高位被忽略了更为准确妥当，同时猜测这种高位被忽略的现象与PC值无关。应该在LDRR4前加入NOP或其他指令来改变LDR R4的Address来继续研究。

除此之外，联系其他文章，不难发现，EMFI的位置、电压、芯片的封装、布局、时钟周期、Jitter均需要考虑在内。

