module fluxfiles

import gui
import fs
import memory
import string
import stdio

const WIDTH = 400
const HEIGHT = 300
const MAX_FILES = 128

var file_list_view
var file_preview
var open_button
var delete_button
var refresh_button
var current_dir = "/"
var selected_file = 0

fn open_window() {
    var win = gui.create_window("FluxFiles", WIDTH, HEIGHT)

    gui.label(win, 10, 10, "Files:")
    file_list_view = gui.list(win, 10, 30, 180, 200, on_file_selected)

    gui.label(win, 200, 10, "Preview:")
    file_preview = gui.text_area(win, 200, 30, 180, 170)

    open_button = gui.button(win, 10, 240, "Open", on_open_clicked)
    delete_button = gui.button(win, 80, 240, "Delete", on_delete_clicked)
    refresh_button = gui.button(win, 150, 240, "Refresh", on_refresh_clicked)

    gui.show_window(win)
    load_files()
}

fn load_files() {
    gui.clear_list(file_list_view)

    var count = fs.count_files(current_dir)
    var i = 0
    while i < count {
        var fname = fs.get_file_name(current_dir, i)
        gui.add_list_item(file_list_view, fname)
        i += 1
    }

    gui.set_text(file_preview, "Select a file to preview.")
}

fn on_file_selected(index, name_ptr) {
    selected_file = name_ptr

    var full_path = memory.alloc(256)
    string.copy(full_path, current_dir)
    string.append(full_path, "/")
    string.append(full_path, name_ptr)

    var contents = fs.read_file(full_path)
    if contents == 0 {
        gui.set_text(file_preview, "(Failed to read file)")
    } else {
        gui.set_text(file_preview, contents)
    }
}

fn on_open_clicked() {
    if selected_file == 0 {
        gui.set_text(file_preview, "No file selected.")
        return
    }

    stdio.print("Opening file: ")
    stdio.print(selected_file)
    stdio.print("\n")

    // TODO: pass to FluxIDE or launcher
}

fn on_delete_clicked() {
    if selected_file == 0 {
        gui.set_text(file_preview, "No file selected.")
        return
    }

    var full_path = memory.alloc(256)
    string.copy(full_path, current_dir)
    string.append(full_path, "/")
    string.append(full_path, selected_file)

    fs.delete_file(full_path)
    load_files()
    gui.set_text(file_preview, "File deleted.")
}

fn on_refresh_clicked() {
    load_files()
}

fn main() {
    open_window()
}