---
description: 'Creator: Ravin'
---

# Find my password!

## Description

{% code overflow="wrap" %}
```
An agent has infiltrated my computer and now ive lost it! There is something unusual about them and it seems like theyve packed and compressed our methods of obtaining anything back. It seemed like a normal file at first, but now it doesnt.... Help me find my password please
```
{% endcode %}

Category: Reverse Engineering (Easy)

## Walkthrough

We are provided with an ELF file, although there isnt much information from the file command

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

Running the strings utility on the executable, we can see that the program was packed using UPX

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

Simple enough, we can unpack the executable using the built-in upx tool in Kali

<figure><img src="../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

Now, we can open the executable in Ghidra to decompile it and recover the password. After decompiling the program, Ghidra provides us with the following pseudo-C code

```c

undefined8 main(void)

{
  int iVar1;
  undefined8 uVar2;
  char *pcVar3;
  undefined8 local_158;
  undefined8 local_150;
  undefined8 local_148;
  undefined5 local_140;
  undefined3 uStack_13b;
  undefined5 uStack_138;
  char local_128 [256];
  size_t local_28;
  char *local_20;
  undefined1 local_11;
  size_t local_10;
  
  local_158 = 0x99f8d19f98ece4e2;
  local_150 = 0xc4eff599d9d899dc;
  local_148 = 0xc49bd89999c49bcd;
  local_140 = 0xf5d9c3f5cd;
  uStack_13b = 0xd0d0cf;
  uStack_138 = 0xd78bd0d0d0;
  local_10 = 0x25;
  local_11 = 0xaa;
  iVar1 = check_debugger();
  if (iVar1 == 0) {
    local_20 = (char *)malloc(local_10 + 1);
    if (local_20 == (char *)0x0) {
      uVar2 = 2;
    }
    else {
      memcpy(local_20,&local_158,local_10);
      xor_buf(local_20,local_10,local_11);
      local_20[local_10] = '\0';
      printf("Enter password: ");
      pcVar3 = fgets(local_128,0x100,stdin);
      if (pcVar3 == (char *)0x0) {
        puts("Input error");
        free(local_20);
        uVar2 = 3;
      }
      else {
        local_28 = strcspn(local_128,"\r\n");
        local_128[local_28] = '\0';
        iVar1 = strcmp(local_128,local_20);
        if (iVar1 == 0) {
          puts("Access granted!");
        }
        else {
          puts("Access denied!");
        }
        memset(local_20,0,local_10);
        free(local_20);
        uVar2 = 0;
      }
    }
  }
  else {
    puts("Debugger detected! Exiting...");
    uVar2 = 1;
  }
  return uVar2;
}


void xor_buf(long param_1,ulong param_2,byte param_3)

{
  undefined8 local_10;
  
  for (local_10 = 0; local_10 < param_2; local_10 = local_10 + 1) {
    *(byte *)(local_10 + param_1) = *(byte *)(local_10 + param_1) ^ param_3;
  }
  return;
}


```

Analyzing the code, we can see that local\_158 and the 3 variables below it appear to contain the hexadecimal value of the XOR'd password, and local\_11 contains the XOR key (local\_10 is the length of the XOR'd password). For some reason, I am not able to get the string to be XOR'd back properly, so we pass the script into ChatGPT to help us deobfuscate and calculate the un-XOR'd password

<figure><img src="../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

## Conclusion

This section will be updated later when I figure out the correct inputs and recipe to get the correct output in CyberChef
