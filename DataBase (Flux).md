module database

// --- Types ---
struct Entry {
    key: str
    value: str
}

// --- Constants ---
const MAX_ENTRIES = 1024

// --- Storage ---
var db: [Entry] = [Entry { key: "", value: "" }] * MAX_ENTRIES
var db_size: int = 0

// --- Set (Insert or Update) ---
fn set(key: str, value: str) {
    for i in 0..db_size {
        if db[i].key == key {
            db[i].value = value
            return
        }
    }
    if db_size < MAX_ENTRIES {
        db[db_size] = Entry { key: key, value: value }
        db_size += 1
    }
}

// --- Get ---
fn get(key: str): str {
    for i in 0..db_size {
        if db[i].key == key {
            return db[i].value
        }
    }
    return "(not found)"
}

// --- Delete ---
fn delete(key: str) {
    for i in 0..db_size {
        if db[i].key == key {
            // Shift entries
            for j in i..(db_size - 1) {
                db[j] = db[j + 1]
            }
            db_size -= 1
            return
        }
    }
}

// --- List all entries ---
fn list() {
    for i in 0..db_size {
        stdio.print("[" + db[i].key + "] = " + db[i].value)
    }
}