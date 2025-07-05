module server

import stdio
import net  // Assumed networking module (provide bindings to bare-metal or OS layer)

const PORT = 8080
const BUFFER_SIZE = 1024

fn start_server() {
    let sock = net.socket(net.AF_INET, net.SOCK_STREAM, 0)
    if sock < 0 {
        stdio.print("Error: Cannot create socket.")
        return
    }

    let addr = net.Address {
        family: net.AF_INET,
        port: PORT,
        ip: "0.0.0.0"
    }

    if net.bind(sock, addr) < 0 {
        stdio.print("Error: Cannot bind to port.")
        return
    }

    if net.listen(sock, 5) < 0 {
        stdio.print("Error: Listen failed.")
        return
    }

    stdio.print("Server running on port " + PORT.str())

    while true {
        let client = net.accept(sock)
        if client < 0 {
            stdio.print("Error: Accept failed.")
            continue
        }

        stdio.print("Client connected!")

        let buffer: [u8] = [0] * BUFFER_SIZE
        let len = net.recv(client, buffer, BUFFER_SIZE)
        if len > 0 {
            let msg = str(buffer[0..len])
            stdio.print("Received: " + msg)

            let reply = "Echo: " + msg
            net.send(client, reply.bytes(), reply.len())
        }

        net.close(client)
    }
}