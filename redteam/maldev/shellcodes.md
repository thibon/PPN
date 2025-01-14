# Shellcodes

- [https://www.ired.team/offensive-security/code-injection-process-injection/writing-and-compiling-shellcode-in-c](https://www.ired.team/offensive-security/code-injection-process-injection/writing-and-compiling-shellcode-in-c)
- [https://www.codeproject.com/Articles/5304605/Creating-Shellcode-from-any-Code-Using-Visual-Stud](https://www.codeproject.com/Articles/5304605/Creating-Shellcode-from-any-Code-Using-Visual-Stud)

Compile runners with a Bash script (like [shcode2exe](https://github.com/accidentalrebel/shcode2exe)):

{% code title="bin2compile.sh" %}
```bash
#!/usr/bin/env bash

# Usage:
#   bin2compile.sh {32|64} <INPUT_BIN> [OUTPUT_EXE]
# Examples:
#   msfvenom -p windows/x64/exec CMD=calc.exe -f raw -o calc.bin
#   bin2compile.sh 64 calc.bin calc.exe

ARCH="${1}"
SC_PATH=`realpath "${2}"`
SC_NAME=`basename "${SC_PATH}"`
SC_NAME="${SC_NAME%.*}"
[[ "${#}" -gt 2 ]] && EXE_NAME="${3}" || EXE_NAME="${SC_NAME}.exe"

cat << EOT > "/tmp/${SC_NAME}.asm"
    global _start
    section .text
_start:
    incbin "${SC_PATH}"
EOT

if [[ "${ARCH}" == "32" ]]; then
    NASM_ARCH="win32"
    LD_ARCH="i386pe"
elif [[ "${ARCH}" == "64" ]]; then
    NASM_ARCH="win64"
    LD_ARCH="i386pep"
fi

echo "[*] Compile time: `date`"
echo "[*] Compiling x${ARCH}"

nasm -f "${NASM_ARCH}" -o "/tmp/${SC_NAME}.obj" "/tmp/${SC_NAME}.asm"
ld -m "${LD_ARCH}" -o "${EXE_NAME}" "/tmp/${SC_NAME}.obj"

if [[ "$?" -ne 1 ]]; then
    echo "[+] Success"
    echo "[+] Output size: `stat -c %s ${EXE_NAME} | numfmt --to=iec`"
else
    echo "[-] Failed"
fi

rm -f /tmp/${SC_NAME}.{asm,obj}
```
{% endcode %}
