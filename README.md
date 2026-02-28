# Binary-Instrumentation-2 Pico-CTF
I've been learning more Windows API functions to do my bidding. Hmm... I swear this program was supposed to create a file and write the flag directly to the file. Can you try and intercept the file writing function to see what went wrong?

# Exploitation

bininst2.exe presents itself as a program that should create a file and write the flag into it, but in practice it never does. The reason only becomes clear when you stop looking at what the binary *does* and start looking at what it *carries*.

The main executable works as a loader its real purpose is to unpack and run a second binary hidden inside it, compressed with LZMA in the .ATOM section. This section is not part of the standard PE format, which is an immediate sign that something was deliberately embedded there. The data starts with the byte 0x5D, which is the LZMA format signature, followed by the dictionary size and uncompressed size fields typical of that algorithm. Decompressing that content yields a complete second Windows executable of 11 KB.

That internal executable is where the actual challenge logic lives. It imports CreateFileA and WriteFile from KERNEL32.dll the functions that should create the file and write the flag. The problem is in the argument passed to CreateFileA: the file path is literally the string <Insert path here>, a placeholder the developer forgot to replace. Since that path is invalid, the function fails silently, returns an invalid handle, and WriteFile never gets to write anything. The flag simply never reaches disk.

What makes the challenge interesting is that the flag did not need to be intercepted at runtime it was already there, embedded as Base64 directly in the internal executable's code, just waiting to be read. Decoding the string reveals the flag immediately.

The entire exploitation comes down to realizing that the binary you are given is not the binary that matters, and that the bug is not in the write logic itself, but in something as simple as a file path that was never filled in.

# Ghidra Analysis

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a7ae5f30-503f-4e26-98be-d1ad6a1a4200" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/6e0cd689-552a-483e-8acc-d2faa4a7f930" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/7f96e4dd-f556-423b-8d45-5bb94b1acf9e" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/cbd3b21f-a9f2-460f-b4ad-11ace110e8c5" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/c630c0f9-44f9-4697-9973-48b92a48ae54" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/9086ca54-e7a4-4d10-b0de-c7d13a44d52d" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a580b25a-e44e-4126-b742-da9c7187c7c7" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/d1b4a13b-ebf9-4dda-ab08-52d64824eb0a" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/9d372837-dba4-469c-a4c3-abe1a5f156b2" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/c905f89b-57bc-4937-95ee-bbc9235f3d3b" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/c0e1b434-0dcd-464c-bd7b-dce4225ebc98" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/da6d6154-07fc-406f-bd50-3570a7dc03a7" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/2ea1cb3e-1fe5-467f-9305-58a67ec63e75" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/881c5992-ee52-4d46-ba01-0ec93f89768c" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/1eaa8390-a21b-48cc-acf9-ae72e05406fb" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/fce9e108-620d-43ea-a072-32de74377424" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/94a5e7d6-480b-42a4-863f-621655a0e000" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a6a0df63-58c0-42e7-942d-5c10c2081b67" />

