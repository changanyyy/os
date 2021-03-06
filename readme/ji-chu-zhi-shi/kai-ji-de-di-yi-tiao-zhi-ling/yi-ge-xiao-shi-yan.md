# ð ä¸ä¸ªå°å®éª

åå¨å®éªæ ¹ç®å½éè¾å¥`make`ï¼ç¶åè¾å¥ï¼ä¸é¢è¿è¡æææ¯ä¸å¼¹çªï¼ï¼

```
$ make qemu-nox-gdb
```

æèï¼ä¸é¢è¿è¡æææ¯æå¼¹çªï¼ï¼

```
$ make qemu-gdb
```

è¿æ¶å°±ä¼å¼å¯`qemu`éç`gdb server`ã

ç¶åå¨å®éªæ ¹ç®å½åæå¼ä¸ä¸ªç»ç«¯ï¼è¾å¥`make gdb`ï¼è§å¯ä¸ä¸ã

{% hint style="info" %}
exercise2ï¼æ ¹æ®ä½ çå°çï¼åç­ä¸é¢é®é¢

æä»¬ä»çè§çé£æ¡æä»¤å¯ä»¥æ¨æ­åºå ç¹ï¼

* çµèå¼æºç¬¬ä¸æ¡æä»¤çå°åæ¯ä»ä¹ï¼è¿ä½äºä»ä¹å°æ¹ï¼
* çµèå¯å¨æ¶`CS`å¯å­å¨å`IP`å¯å­å¨çå¼æ¯ä»ä¹ï¼
* ç¬¬ä¸æ¡æä»¤æ¯ä»ä¹ï¼ä¸ºä»ä¹è¿æ ·è®¾è®¡ï¼ï¼åé¢æè§£éï¼ç¨èªå·±è¯ç®è¿°ï¼
{% endhint %}

### QEMUä¸ºä»ä¹ä¼è¿æ ·å¯å¨å¢?

`Intel`å°±æ¯è¿æ ·è®¾è®¡`8088`å¤çå¨çï¼ä¸åCPUæ¶æä¸ï¼BIOSå¯è½ä¸ä¸æ ·ï¼è¿éåªæ¯ä¸¾äºx86æ¶æçä¾å­ï¼ã

å ä¸ºçµèç`BIOS`æ¯âå¤©ççâç©çå°åèå´`0x000f0000-0x000fffff`ï¼è¿ç§è®¾è®¡å¯ä»¥ç¡®ä¿æºå¨ç`BIOS`æ»æ¯å¨å¼æºä¹ååè·å¾æºå¨æ§å¶æï¼ä½ç½®åºå®ï¼ã`QEMU`æ¨¡æå¨èªå¸¦èªå·±ç`BIOS`ï¼å®å°`BIOS`æ¾ç½®å¨å¤çå¨æ¨¡æç©çå°åç©ºé´çè¿ä¸ªä½ç½®ãå¤çå¨å¤ä½æ¶ï¼ï¼æ¨¡æï¼å¤çå¨è¿å¥å®æ¨¡å¼ï¼å¹¶å°`CS`è®¾ç½®ä¸º`0xf000`, `IP`è®¾ç½®ä¸º`0xfff0`ã

é£ä¹åæ®µå°å`0xf000:fff0`å¦ä½åæç©çå°åï¼

ä¸ºäºåç­è¿ä¸ªé®é¢ï¼æä»¬éè¦äºè§£ä¸äºå³äºå®éæ¨¡å¼å¯»åçç¥è¯ãå¨å®æ¨¡å¼ä¸(å³`PC`åå¯å¨çæ¨¡å¼)ï¼å°åè½¬æ¢çå·¥ä½å¬å¼ä¸º:`ç©çå°å= 16 *æ®µ+åç§»é`ã

å æ­¤ï¼å½è®¡ç®æºå°`CS`è®¾ç½®ä¸º`0xf000`, `IP`è®¾ç½®ä¸º`0xfff0`æ¶ï¼æå¼ç¨çç©çå°åä¸º:

```c
  16 * 0xf000 + 0xfff0   # in hex multiplication by 16 is
   = 0xf0000 + 0xfff0    # easy--just append a 0.
   = 0xffff0 
```

`0xffff0`æ¯`BIOS`ç»æå16ä¸ªå­è(`0x100000`)ãå¨ç¬¬ä¸æ¡æä»¤çåé¢åªæ16ä¸ªå­èï¼å¥ä¹å¹²ä¸äºãæä»¥ï¼......

{% hint style="info" %}
exercise3ï¼è¯·ç¿»éæ ¹ç®å½ä¸ç`makefile`æä»¶ï¼ç®è¿°`make qemu-nox-gdb`å`make gdb`æ¯æä¹è¿è¡çï¼`.gdbinit`æ¯`gdb`åå§åæä»¶ï¼äºè§£å³å¯ï¼
{% endhint %}
