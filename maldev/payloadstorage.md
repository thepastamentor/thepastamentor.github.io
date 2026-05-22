---
layout: default
title: Payload Storage
parent: Maldev Resources
nav_order: 2
---

# Payload Storage

Payloads can be stored in `.text`, `.data`, or `.rsrc`
`.text` stored payloads are executable by default, `.data` and `.rsrc` stored payloads require you to make the memory executable at runtime.
## `.text`

Put the payload into a function e.g. main function. 
```c
int main(int argc, char *argv[]){

	unsigned char shellcode[] = {
		// shellcode here
	};
	
	// everything else main does
} 
```
## `.data`

Tell the compiler that your payload is read only data, accomplished by creating a global variable holding your payload.
```c
unsigned char shellcode[] = {
	// shellcode here
};
unsigned int shellcode_len = <shellcode_len>;

int main(int argc, char *argv[]){
  // main contents
} 
```
## `.rsrc`

Create a small file holding your payload and tell the compiler it should be part of the resources section, use API to reach into the resources section and extract the payload from there.
In the folder, have your payload in a binary format, in this case it would be `calc.ico`.

`main.c`
```c
#include "resources.h"

int main(int argc, char *argv[]){
	HRSRC res;
	HGLOBAL resHandle;
	unsigned char * payload;
	unsigned int payload_len;
	
	res = FindResource(NULL, MAKEINTRESOURCE(FAVICON_ICO), RT_RCDATA); // return handle of FAVICON_ICO resource
	resHandle = LoadResource(NULL, res); // returns handle to module containing our resource
	payload = (char *) LockResource(resHandle); // returns pointer to first byte of our payload
	payload_len = SizeofResource(NULL,res); // returns the size of the payload

	// everything else main does
} 
```

`resources.h`
```c
#define FAVICON_ICO 100
```

`resources.rc`
```c
#include "resources.h"

FAVICON_ICO RCDATA calc.ico
```