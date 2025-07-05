// ----------------------------------------
// File: framework/fluxfx.flux
// ----------------------------------------
module fluxfx

import gui
import stdio
import fs
import string

// ----------------------
// Type: App
// ----------------------
type App = struct {
  title: string
  init: fn()
}

// ----------------------
// App Registry
// ----------------------
var app_list: [32]App
var app_count = 0

fn register_app(title: string, init_fn: fn()) {
  app_list[app_count].title = title
  app_list[app_count].init = init_fn
  app_count = app_count + 1
}

fn launch_app(index: int) {
  var app = app_list[index]
  stdio.print("Launching: ")
  stdio.print(app.title)
  stdio.print("\n")
  app.init()
}

// ----------------------
// Logging
// ----------------------
fn log(msg: string) {
  stdio.print("[FluxFX] ")
  stdio.print(msg)
  stdio.print("\n")
}

// ----------------------
// Alert Dialog
// ----------------------
fn alert(title: string, message: string) {
  var win = gui.create_window(title, 240, 120)
  gui.label(win, 10, 10, message)
  gui.button(win, 80, 70, "OK", fn() {})
  gui.show_window(win)
}

// ----------------------
// Prompt Input Dialog
// ----------------------
fn prompt(title: string, label: string) -> string {
  var result = ""
  var win = gui.create_window(title, 300, 160)
  gui.label(win, 10, 10, label)
  var inputbox = gui.input(win, 10, 40, 280)

  gui.button(win, 10, 80, "OK", fn() {
    result = gui.get_text(inputbox)
  })

  gui.show_window(win)
  return result
}

// ----------------------
// File Save Dialog
// ----------------------
fn save_dialog(default_path: string, contents: string) {
  var win = gui.create_window("Save File", 300, 160)
  gui.label(win, 10, 10, "Save as:")
  var inputbox = gui.input(win, 10, 40, 280)
  gui.set_text(inputbox, default_path)

  gui.button(win, 10, 80, "Save", fn() {
    var path = gui.get_text(inputbox)
    fs.write_file(path, contents)
    log("Saved to " + path)
  })

  gui.show_window(win)
}

// ----------------------
// File Open Dialog
// ----------------------
fn open_dialog(ext: string, callback: fn(string)) {
  var path = fs.prompt_open(ext)
  var contents = fs.read_file(path)
  callback(contents)
}

// ----------------------
// Home Screen (App Menu)
// ----------------------
fn home_screen() {
  var win = gui.create_window("FluxFX Home", 300, 40 + app_count * 40)
  gui.label(win, 10, 10, "Applications")

  var y = 40
  var i = 0
  while i < app_count {
    var app = app_list[i]
    var idx = i
    gui.button(win, 10, y, app.title, fn() {
      launch_app(idx)
    })
    y = y + 30
    i = i + 1
  }

  gui.show_window(win)
}

// ----------------------
// Debug Console (Optional)
// ----------------------
fn console_window() {
  var win = gui.create_window("Flux Console", 500, 300)
  var output = gui.text_area(win, 10, 30, 480, 230)
  var input = gui.input(win, 10, 270, 400)

  gui.button(win, 420, 270, "Send", fn() {
    var cmd = gui.get_text(input)
    gui.append_text(output, "> " + cmd + "\n")
  })

  gui.show_window(win)
}