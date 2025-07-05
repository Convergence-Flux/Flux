module stack

import memory
import stdio

type Stack = struct {
    data_ptr
    capacity
    count
}

fn create_stack(initial_capacity) -> Stack {
    var stack
    stack.data_ptr = memory.alloc(initial_capacity)
    stack.capacity = initial_capacity
    stack.count = 0
    return stack
}

fn push(stack, value) {
    if stack.count >= stack.capacity {
        // Resize
        var new_capacity = stack.capacity * 2
        var new_data = memory.alloc(new_capacity)
        var i = 0
        while i < stack.count {
            new_data[i] = stack.data_ptr[i]
            i += 1
        }
        memory.free(stack.data_ptr)
        stack.data_ptr = new_data
        stack.capacity = new_capacity
    }

    stack.data_ptr[stack.count] = value
    stack.count += 1
}

fn pop(stack) -> int {
    if stack.count == 0 {
        stdio.print("Stack underflow!\n")
        return -1
    }
    stack.count -= 1
    return stack.data_ptr[stack.count]
}

fn peek(stack) -> int {
    if stack.count == 0 {
        stdio.print("Stack is empty!\n")
        return -1
    }
    return stack.data_ptr[stack.count - 1]
}

fn is_empty(stack) -> bool {
    return stack.count == 0
}

fn length(stack) -> int {
    return stack.count
}