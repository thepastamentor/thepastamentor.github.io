---
layout: default
title: Windows Data Types
parent: Maldev Resources
nav_order: 1
---

# Windows Data Types

A quick reference sheet of Windows data types and their format specifiers.

For `BOOL`, don't compare `== TRUE`, as its an `int` — compare `!= FALSE` or just use the value directly since any non-zero value is truthy.

<style>
.dt-wrap { overflow-x: auto; width: 100%; }
.dt-wrap table { border-collapse: collapse; font-size: 0.8rem; white-space: nowrap; width: max-content; }
.dt-wrap th { text-align: left; padding: 6px 12px; border-bottom: 2px solid #444; }
.dt-wrap td { padding: 5px 12px; border-bottom: 1px solid #333; vertical-align: top; white-space: normal; max-width: 320px; }
.dt-wrap td:nth-child(1), .dt-wrap td:nth-child(5), .dt-wrap td:nth-child(6) { white-space: nowrap; }
.dt-wrap code { font-size: 0.78rem; }
</style>

<div class="dt-wrap">
<table>
<thead>
<tr><th>Type</th><th>Category</th><th>Underlying C type</th><th>Bits</th><th>printf (ANSI)</th><th>wprintf (Unicode)</th><th>Notes</th></tr>
</thead>
<tbody>
<tr><td><code>BYTE</code></td><td>Unsigned int</td><td><code>unsigned char</code></td><td>8</td><td><code>%u</code> or <code>%hhu</code></td><td><code>%u</code> or <code>%hhu</code></td><td>Often used for raw byte buffers.</td></tr>
<tr><td><code>CHAR</code></td><td>Integer</td><td><code>char</code></td><td>8</td><td><code>%c</code></td><td><code>%c</code></td><td>ANSI character. Use <code>%s</code> for strings.</td></tr>
<tr><td><code>SHORT</code></td><td>Integer</td><td><code>short</code></td><td>16</td><td><code>%hd</code></td><td><code>%hd</code></td><td>Signed 16-bit integer.</td></tr>
<tr><td><code>WORD</code></td><td>Unsigned int</td><td><code>unsigned short</code></td><td>16</td><td><code>%hu</code></td><td><code>%hu</code></td><td>Unsigned 16-bit. Common in struct fields.</td></tr>
<tr><td><code>INT</code></td><td>Integer</td><td><code>int</code></td><td>32</td><td><code>%d</code> or <code>%i</code></td><td><code>%d</code> or <code>%i</code></td><td>Standard signed 32-bit integer.</td></tr>
<tr><td><code>UINT</code></td><td>Unsigned int</td><td><code>unsigned int</code></td><td>32</td><td><code>%u</code></td><td><code>%u</code></td><td>Unsigned 32-bit. Same width as DWORD.</td></tr>
<tr><td><code>LONG</code></td><td>Integer</td><td><code>long</code></td><td>32</td><td><code>%ld</code></td><td><code>%ld</code></td><td>Always 32-bit on Windows, unlike Linux.</td></tr>
<tr><td><code>ULONG</code></td><td>Unsigned int</td><td><code>unsigned long</code></td><td>32</td><td><code>%lu</code></td><td><code>%lu</code></td><td>Unsigned 32-bit. Very common in WinAPI.</td></tr>
<tr><td><code>DWORD</code></td><td>Unsigned int</td><td><code>unsigned long</code></td><td>32</td><td><code>%lu</code> or <code>%08lX</code></td><td><code>%lu</code> or <code>%08lX</code></td><td>Workhorse unsigned 32-bit. Hex with <code>%lX</code>.</td></tr>
<tr><td><code>DWORD32</code></td><td>Unsigned int</td><td><code>unsigned int</code></td><td>32</td><td><code>%u</code></td><td><code>%u</code></td><td>Explicit 32-bit, always UINT underneath.</td></tr>
<tr><td><code>DWORD64</code></td><td>Unsigned int</td><td><code>unsigned __int64</code></td><td>64</td><td><code>%I64u</code></td><td><code>%I64u</code></td><td>Use <code>%llu</code> on non-MSVC or with <code>_USE_32BIT_TIME_T</code> unset.</td></tr>
<tr><td><code>LONGLONG</code></td><td>Integer</td><td><code>__int64</code></td><td>64</td><td><code>%I64d</code></td><td><code>%I64d</code></td><td>Also valid: <code>PRId64</code> with <code>&lt;inttypes.h&gt;</code>.</td></tr>
<tr><td><code>ULONGLONG</code></td><td>Unsigned int</td><td><code>unsigned __int64</code></td><td>64</td><td><code>%I64u</code></td><td><code>%I64u</code></td><td>Also valid: <code>PRIu64</code>. Used by FILETIME math.</td></tr>
<tr><td><code>INT_PTR</code></td><td>Pointer / handle</td><td><code>__int64</code> / <code>int</code></td><td>arch</td><td><code>%Id</code></td><td><code>%Id</code></td><td>Signed, pointer-sized. 64-bit on x64, 32-bit on x86.</td></tr>
<tr><td><code>UINT_PTR</code></td><td>Pointer / handle</td><td><code>unsigned __int64</code> / <code>unsigned int</code></td><td>arch</td><td><code>%Iu</code></td><td><code>%Iu</code></td><td>Unsigned pointer-sized integer.</td></tr>
<tr><td><code>LONG_PTR</code></td><td>Pointer / handle</td><td><code>__int64</code> / <code>long</code></td><td>arch</td><td><code>%Id</code></td><td><code>%Id</code></td><td>Pointer-sized signed long.</td></tr>
<tr><td><code>ULONG_PTR</code></td><td>Pointer / handle</td><td><code>unsigned __int64</code> / <code>unsigned long</code></td><td>arch</td><td><code>%Iu</code></td><td><code>%Iu</code></td><td>Pointer-sized unsigned long. Param for WndProc.</td></tr>
<tr><td><code>SIZE_T</code></td><td>Unsigned int</td><td><code>ULONG_PTR</code></td><td>arch</td><td><code>%Iu</code> or <code>%zu</code></td><td><code>%Iu</code> or <code>%zu</code></td><td>Result of <code>sizeof()</code>. <code>%zu</code> works in C99+ MSVC.</td></tr>
<tr><td><code>SSIZE_T</code></td><td>Integer</td><td><code>LONG_PTR</code></td><td>arch</td><td><code>%Id</code></td><td><code>%Id</code></td><td>Signed SIZE_T.</td></tr>
<tr><td><code>PVOID</code> / <code>LPVOID</code></td><td>Pointer / handle</td><td><code>void *</code></td><td>arch</td><td><code>%p</code></td><td><code>%p</code></td><td>Generic pointer. <code>%p</code> prints address.</td></tr>
<tr><td><code>HANDLE</code></td><td>Pointer / handle</td><td><code>void *</code></td><td>arch</td><td><code>%p</code></td><td><code>%p</code></td><td>Opaque handle. Print address, not value.</td></tr>
<tr><td><code>HWND</code></td><td>Pointer / handle</td><td><code>HANDLE</code></td><td>arch</td><td><code>%p</code></td><td><code>%p</code></td><td>Window handle.</td></tr>
<tr><td><code>HMODULE</code> / <code>HINSTANCE</code></td><td>Pointer / handle</td><td><code>HANDLE</code></td><td>arch</td><td><code>%p</code></td><td><code>%p</code></td><td>Module handle. Treat as pointer.</td></tr>
<tr><td><code>FARPROC</code></td><td>Pointer / handle</td><td><code>function pointer</code></td><td>arch</td><td><code>%p</code></td><td><code>%p</code></td><td>Cast to <code>(void *)</code> before printing.</td></tr>
<tr><td><code>CHAR</code> / <code>LPSTR</code> / <code>LPCSTR</code></td><td>String</td><td><code>char *</code> / <code>const char *</code></td><td>8</td><td><code>%s</code></td><td><code>%hs</code> or <code>%S</code> (deprecated)</td><td>ANSI string. In wprintf, <code>%hs</code> is safe; <code>%S</code> is legacy.</td></tr>
<tr><td><code>WCHAR</code> / <code>LPWSTR</code> / <code>LPCWSTR</code></td><td>String</td><td><code>wchar_t *</code> / <code>const wchar_t *</code></td><td>16</td><td><code>%ls</code> or <code>%S</code> (MSVC)</td><td><code>%s</code> or <code>%ls</code></td><td>Wide string. In wprintf: <code>%s</code>. In printf: <code>%ls</code>. <code>L"..."</code> literals.</td></tr>
<tr><td><code>TCHAR</code> / <code>LPTSTR</code> / <code>LPCTSTR</code></td><td>String</td><td><code>WCHAR</code> or <code>CHAR</code> (UNICODE macro)</td><td>arch</td><td><code>%s</code> or <code>%ls</code></td><td><code>%s</code> or <code>%ls</code></td><td>Use <code>_T()</code> / <code>TEXT()</code> macros. Expands to WCHAR if UNICODE defined.</td></tr>
<tr><td><code>BSTR</code></td><td>String</td><td><code>WCHAR *</code> (length-prefixed)</td><td>16</td><td><code>%ls</code></td><td><code>%s</code></td><td>COM string. Use <code>SysAllocString</code> / <code>SysFreeString</code>.</td></tr>
<tr><td><code>BOOL</code></td><td>Bool / status</td><td><code>int</code></td><td>32</td><td><code>%d</code></td><td><code>%d</code></td><td>TRUE=1, FALSE=0. NOT a C99 bool. Check <code>!= FALSE</code>.</td></tr>
<tr><td><code>BOOLEAN</code></td><td>Bool / status</td><td><code>BYTE</code> (unsigned char)</td><td>8</td><td><code>%u</code></td><td><code>%u</code></td><td>Used in NT native API. 0=false, non-0=true.</td></tr>
<tr><td><code>HRESULT</code></td><td>Bool / status</td><td><code>LONG</code> (signed)</td><td>32</td><td><code>0x%08lX</code></td><td><code>0x%08lX</code></td><td>Always print as hex. <code>SUCCEEDED()</code> / <code>FAILED()</code> macros.</td></tr>
<tr><td><code>NTSTATUS</code></td><td>Bool / status</td><td><code>LONG</code> (signed)</td><td>32</td><td><code>0x%08lX</code></td><td><code>0x%08lX</code></td><td>NT status code. <code>NT_SUCCESS()</code> macro.</td></tr>
<tr><td><code>FLOAT</code></td><td>Float / misc</td><td><code>float</code></td><td>32</td><td><code>%f</code> or <code>%g</code></td><td><code>%f</code> or <code>%g</code></td><td>Single-precision. Same as standard C float.</td></tr>
<tr><td><code>DOUBLE</code></td><td>Float / misc</td><td><code>double</code></td><td>64</td><td><code>%lf</code> or <code>%g</code></td><td><code>%lf</code> or <code>%g</code></td><td>Double-precision. <code>%g</code> trims trailing zeros.</td></tr>
<tr><td><code>FILETIME</code></td><td>Float / misc</td><td><code>struct { DWORD dwLowDateTime; DWORD dwHighDateTime; }</code></td><td>64</td><td><code>%I64u</code> (combine first)</td><td><code>%I64u</code> (combine first)</td><td>100ns intervals since Jan 1 1601. Cast: <code>(ULONGLONG)ft.dwHighDateTime&lt;&lt;32 | ft.dwLowDateTime</code>.</td></tr>
<tr><td><code>GUID</code> / <code>UUID</code></td><td>Float / misc</td><td><code>struct</code> (16 bytes)</td><td>128</td><td><code>StringFromGUID2</code> or manual <code>sprintf</code></td><td><code>StringFromGUID2</code> or <code>swprintf</code></td><td>Print each field individually or use RPC helpers.</td></tr>
<tr><td><code>COLORREF</code></td><td>Float / misc</td><td><code>DWORD</code></td><td>32</td><td><code>RGB(%u,%u,%u)</code> or <code>0x%06lX</code></td><td><code>0x%06lX</code></td><td>Packed RGB: low byte = red. Use <code>GetRValue</code> / <code>GetGValue</code> / <code>GetBValue</code>.</td></tr>
</tbody>
</table>
</div>