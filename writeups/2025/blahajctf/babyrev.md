---
description: 'Creator: scuffed'
---

# babyrev

## Description

```
Use your favourite decompiler or debugger to solve this challenge!
```

Category: Reverse Engineering (Easy)

## Walkthrough

We are provided with a Linux executable, which we can disassemble in Ghidra to view a pseudo-C approximation of its assembly code.

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

```c
undefined8 main(void)

{
  int iVar1;
  long in_FS_OFFSET;
  int local_4c;
  char local_48 [56];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("The crypt opens...");
  for (local_4c = 0; local_4c < 0x29; local_4c = local_4c + 1) {
    flag[local_4c] = flag[local_4c] ^ 0x69;
    print_progress(((local_4c + 1) * 100) / 0x29);
    usleep(50000);
  }
  do {
    printf("\n\nSphinx of black quartz, judge my vow: ");
    __isoc23_scanf(&DAT_00102051,local_48);
    iVar1 = strcmp(local_48,flag);
    if (iVar1 != 0) {
      printf("\nThink harder about your response, child.");
    }
    iVar1 = strcmp(local_48,flag);
  } while (iVar1 != 0);
  printf("\nYou may enter my domain! Your flag: %s.\n",flag);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}

```

The program will read the flag variable, and XOR each character with a hex value 0x69, before updating the current progress bar. We can easily reverse this encryption by extracting the flag variable, and then XOR'ing each character by ourselves

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

```py
ciphertext = ['0b', '05', '08', '01', '08', '03', '12', '1e', '5a', '05', '0a', '59',
'04', '5d', '36', '5e', '59', '36', '1b', '5a', '1f', '5a', '1b', '5c',
'5a', '36', '5a', '07', '50', '58', '07', '5a', '5a', '1b', '58', '07',
'50', '48', '48', '48', '14']

cipher_bytes = [int(c,16) for c in ciphertext]
key = 0x69
plaintext = ''.join(chr(b^key) for b in cipher_bytes)
print(plaintext)
```

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>
