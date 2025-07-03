module bootloader

org 0x7C00     # BIOS loads MBR to 0x7C00

label start:
    cli
    xor ax, ax
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0x7C00
    sti

    ; Show "Loading Flux..." on screen
    mov si, msg_loading
.print:
    lodsb
    cmp al, 0
    je .done
    mov ah, 0x0E
    mov bh, 0
    mov bl, 0x07
    int 0x10
    jmp .print
.done:

    ; Load 10 sectors from disk (sector 2–11) to address 0x1000
    mov bx, 0x1000     # buffer location
    mov dh, 0          # head
    mov dl, 0x00       # drive number (floppy=0x00, HDD=0x80)
    mov ch, 0          # cylinder
    mov cl, 2          # starting sector (sector 1 is MBR)
    mov al, 10         # number of sectors to read
    call disk_read

    jmp 0x0000:0x1000  # jump to kernel

label hang:
    jmp hang

label disk_read:
    mov ah, 0x02       # BIOS disk read
    int 0x13
    jc disk_fail
    ret

label disk_fail:
    mov si, msg_error
    call print_string
    jmp hang

label print_string:
.char:
    lodsb
    cmp al, 0
    je .done
    mov ah, 0x0E
    mov bh, 0
    mov bl, 0x04
    int 0x10
    jmp .char
.done:
    ret

# --- String Messages ---
label msg_loading:
    db "Loading Flux Kernel...", 0

label msg_error:
    db "Disk read error!", 0

# --- Pad to 510 bytes ---
pad 510 - (here - 0x7C00), 0x00

# --- Boot Signature (required) ---
dw 0xAA55