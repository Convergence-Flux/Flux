module miniweb

import gui
import memory
import string
import stdio
import net  // placeholder for future HTTP

const WIDTH = 320
const HEIGHT = 240

var url_input
var page_view
var back_button
var go_button
var history = []

fn open_window() {
    var win = gui.create_window("MiniWeb", WIDTH, HEIGHT)

    gui.label(win, 10, 10, "URL:")
    url_input = gui.input(win, 50, 10, 200)

    go_button = gui.button(win, 260, 10, "Go", on_go_clicked)
    back_button = gui.button(win, 10, 40, "Back", on_back_clicked)

    page_view = gui.text_area(win, 10, 70, 300, 150)
    gui.set_text(page_view, "Welcome to MiniWeb in Flux!\nEnter a URL and press Go.")

    gui.show_window(win)
}

fn on_go_clicked() {
    var url = gui.get_text(url_input)
    if url == 0 || string.length(url) == 0 {
        gui.set_text(page_view, "Please enter a valid URL.")
        return
    }

    history_push(url)

    var html = fetch_stub(url)
    render_page(html)
}

fn on_back_clicked() {
    var previous = history_pop()
    if previous == 0 {
        gui.set_text(page_view, "No history.")
        return
    }

    gui.set_text(url_input, previous)
    var html = fetch_stub(previous)
    render_page(html)
}

fn fetch_stub(url) -> string {
    // Simulate responses to a few known URLs
    if string.equals(url, "http://flux.dev") {
        return "<h1>Flux</h1><p>A self-hosting bare-metal language.</p>"
    }

    if string.equals(url, "http://about.flux") {
        return "<h1>About Flux</h1><p>Flux is minimal, fast, and self-reliant.</p>"
    }

    return "<h1>404</h1><p>Page not found.</p>"
}

fn render_page(html) {
    var output = parse_html_stub(html)
    gui.set_text(page_view, output)
}

fn parse_html_stub(html) -> string {
    // Extremely basic tag stripper
    var out = memory.alloc(1024)
    var i = 0
    var j = 0
    var inside = false

    while html[i] != 0 {
        if html[i] == '<' {
            inside = true
            i += 1
            continue
        }
        if html[i] == '>' {
            inside = false
            i += 1
            continue
        }
        if !inside {
            out[j] = html[i]
            j += 1
        }
        i += 1
    }

    out[j] = 0
    return out
}

fn history_push(url) {
    // Store history in reverse stack
    history.append(url)
}

fn history_pop() -> string {
    if history.length < 2 {
        return 0
    }
    history.pop()  // current
    return history.last()
}

fn main() {
    open_window()
}