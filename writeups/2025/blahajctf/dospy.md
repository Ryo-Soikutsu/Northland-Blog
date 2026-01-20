---
description: 'Creator: scuffed'
---

# doSpy

## Description

```
I wrote this Minesweeper game in .NET, I bet you can't beat it!
```

Category: Reverse Engineering (Easy)

## Walkthrough

We are provided with a 7z archive file, which contains a single executable <mark style="color:red;">DecompileMe.exe</mark>. As the challenge description states, this is a .NET/C# executable, which means that Ghidra may not be very effective in decompiling this program. Instead, we can use a tool called [ILSpy](https://github.com/icsharpcode/ILSpy), a .NET decompiler that we can run on Windows. After installing the decompiler, we can open up the executable in the program to see its inner workings.

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption><p>After importing DecompileMe.exe</p></figcaption></figure>

Looking through the source code, we notice several interesting things in the code. Firstly, the code contains an encrypted flag, which the program will supposedly decrypt. Next, there is a function labelled  DecryptAndShowFlag(), which will use the encrypted flag string we saw earlier.

```csharp
private static void DecryptAndShowFlag()
	{
		Console.WriteLine("\nCongratulations! You've cleared all the safe squares!");
		List<int> list = (from p in MineLocations
			select p.x + p.y * 10 into k
			orderby k
			select k).ToList();
		try
		{
			byte[] array = (from x in Enumerable.Range(0, _encryptedFlag.Length)
				where x % 2 == 0
				select Convert.ToByte(_encryptedFlag.Substring(x, 2), 16)).ToArray();
			StringBuilder stringBuilder = new StringBuilder();
			for (int i = 0; i < array.Length; i++)
			{
				stringBuilder.Append((char)(array[i] ^ list[i % list.Count]));
			}
			Console.WriteLine($"Here is your flag: {stringBuilder}");
		}
		catch (Exception)
		{
			Console.WriteLine("...but the flag decryption failed! An error occurred.");
			Console.WriteLine("You failed!");
		}
	}
```

The function uses a hardcoded list of mine locations to calculate the individual XOR keys for the flag, and then loops through each character in the flag and XORs it with the corresponding key, before printing out the decrypted flag. With all this information, we can write a decrypt script and get our flag

```python
flag = "626e646773796f4666427971777c1d4a157b751b66036e435b0f0d0d746372700d76673a"

coordinates = [(0, 0),
(0, 2),
(0, 3),
(0, 4),
(0, 6),
(0, 7),
(0, 8),
(1, 2),
(1, 5),
(1, 7),
(1, 9),
(2, 0),
(2, 2),
(2, 3),
(2, 4),
(2, 5),
(2, 8),
(3, 2),
(3, 6),
(3, 7),
(3, 8),
(4, 4),
(4, 8),
(5, 0),
(5, 1),
(5, 3),
(5, 6),
(5, 8),
(5, 9),
(6, 3),
(6, 4),
(6, 5),
(6, 7),
(7, 3),
(7, 5),
(7, 6),
(7, 7),
(7, 9),
(8, 1),
(8, 3),
(8, 4),
(8, 5),
(8, 7),
(8, 9),
(9, 1),
(9, 2),
(9, 4),
(9, 5),
(9, 6),
(9, 8)]

key_array = sorted([coord[0] + coord[1]*10 for coord in coordinates])
encrypted_bytes = bytes(int(flag[i:i+2], 16) for i in range(0, len(flag), 2))
decoded = ""
for i, b in enumerate(encrypted_bytes):
    decoded += chr(b ^ key_array[i % len(key_array)])

print(decoded)

```

(It's not very pretty, but its functional and thats all that matters)

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>
