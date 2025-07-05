module fluxide

import gui
import memory
import string
import stdio
import compiler  // hypothetical module to compile/run Flux code
import fs        // file system interface

const WIDTH = 640
const HEIGHT = 400

var code_editor
var output_console
var run_button
var open_button
var current_filename = 0

fn open_window() {
    var win = gui.create_window("FluxIDE", WIDTH, HEIGHT)

    gui.label(win, 10, 10, "Code Editor:")
    code_editor = gui.text_area(win, 10, 30, 620, 200)

    gui.label(win, 10, 240, "Console:")
    output_console = gui.text_area(win, 10, 260, 620, 100)
    gui.set_text(output_console, "Welcome to FluxIDE\n")

    run_button = gui.button(win, 500, 365, "Run", on_run_clicked)
    open_button = gui.button(win, 10, 365, "Open", on_open_clicked)

    gui.show_window(win)
}

fn on_open_clicked() {
    var filename = fs.prompt_open(".flux")
    if filename == 0 {
        gui.set_text(output_console, "Open cancelled.\n")
        return
    }

    current_filename = filename

    var contents = fs.read_file(filename)
    if contents == 0 {
        gui.set_text(output_console, "Failed to read file.\n")
        return
    }

    gui.set_text(code_editor, contents)
    gui.set_text(output_console, "Opened file: ")
    gui.append_text(output_console, filename)
    gui.append_text(output_console, "\n")
}

fn on_run_clicked() {
    var code = gui.get_text(code_editor)
    if code == 0 || string.length(code) == 0 {
        gui.set_text(output_console, "No code to run.\n")
        return
    }

    gui.set_text(output_console, "Compiling...\n")
    var result = compiler.compile_and_run(code)

    gui.append_text(output_console, result)
}

fn main() {
    open_window()
}