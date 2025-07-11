module intelligence_gui

// I/O Ports
const OUT_PORT = 0x01
const IN_PORT = 0x02
const CLEAR_PORT = 0x03
const SLEEP_PORT = 0xFE

const VERSION = "0.1.0"

// Entry
fn main() {
    loop {
        clear_screen()
        draw_gui()
        let choice = get_key()
        handle_choice(choice)
    }
}

// Draw GUI Screen
fn draw_gui() {
    draw_border()
    draw_title("Intelligence OS")
    draw_buttons()
    draw_footer("Press 1–6 to enter a module")
}

// Draw outer border
fn draw_border() {
    print("╔════════════════════════════════════════════════════╗\n")
    let i = 0
    while i < 20 {
        print("║                                                    ║\n")
        i += 1
    }
    print("╚════════════════════════════════════════════════════╝\n")
}

// Centered title
fn draw_title(title) {
    move_cursor(2, 25)
    print("[ ")
    print(title)
    print(" ]\n")
    move_cursor(3, 22)
    print("Version: ")
    print(VERSION)
    print("\n")
}

// Draw buttons
fn draw_buttons() {
    move_cursor(6, 10); print("[1] Supply Chain")
    move_cursor(8, 10); print("[2] Human Resources")
    move_cursor(10, 10); print("[3] Marketing")
    move_cursor(6, 35); print("[4] Accounting")
    move_cursor(8, 35); print("[5] Finance")
    move_cursor(10, 35); print("[6] Economics")
}

// Footer instructions
fn draw_footer(msg) {
    move_cursor(18, 10)
    print(msg)
}

// Respond to input
fn handle_choice(choice) {
    clear_screen()
    if choice == '1' { launch_module("Supply Chain") }
    elif choice == '2' { launch_module("Human Resources") }
    elif choice == '3' { launch_module("Marketing") }
    elif choice == '4' { launch_module("Accounting") }
    elif choice == '5' { launch_module("Finance") }
    elif choice == '6' { launch_module("Economics") }
    else {
        print("Invalid option.\n")
        sleep(1000)
    }
}

// Module screen
fn launch_module(name) {
    draw_border()
    move_cursor(2, 22)
    print("=== ")
    print(name)
    print(" ===\n")
    move_cursor(5, 10)
    print("This module is not implemented yet.\n")
    move_cursor(7, 10)
    print("Press any key to return to the GUI.")
    get_key()
}

// -- BIOS I/O Primitives --

fn clear_screen() {
    out(CLEAR_PORT, 1)
}

fn print(str) {
    let i = 0
    while str[i] != 0 {
        out(OUT_PORT, str[i])
        i += 1
    }
}

fn get_key() -> byte {
    return in(IN_PORT)
}

fn sleep(ms) {
    out(SLEEP_PORT, ms)
}

// Move cursor to row, col (BIOS simulation — optional implementation)
fn move_cursor(row, col) {
    // Print newline until we reach row
    let r = 0
    while r < row {
        print("\n")
        r += 1
    }
    // Print spaces until we reach column
    let c = 0
    while c < col {
        print(" ")
        c += 1
    }
}