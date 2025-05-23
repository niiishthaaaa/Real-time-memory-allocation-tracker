# Real-time-memory-allocation-tracker
import numpy as np
import matplotlib.pyplot as plt
import ipywidgets as widgets
from IPython.display import display, clear_output

memory = np.zeros(16)
allocations = []
mode = "paging"
pid_counter = 0

def plot_memory():
    plt.figure(figsize=(12, 3))
    bars = plt.bar(range(16), memory,
                   color=['#90EE90' if x == 0 else '#FF6B6B' for x in memory],
                   edgecolor='black')

    for start, size, pid in allocations:
        if mode == "segmentation":
            plt.text(start + size/2 - 0.5, 1.2, f"P{pid} ({size})",
                    ha="center", bbox=dict(facecolor='white', alpha=0.7))
        else:
            plt.text(start, 1.2, f"P{pid}",
                    ha="center", bbox=dict(facecolor='white', alpha=0.7))

    plt.title(f"Memory Allocation - {mode.capitalize()} Mode")
    plt.xlabel("Memory Block")
    plt.ylabel("State (0=Free, 1=Used)")
    plt.ylim(0, 1.5)
    plt.grid(axis='x', linestyle='--', alpha=0.3)
    plt.show()

    utilization = np.sum(memory) / 16 * 100
    free_blocks = np.sum(memory == 0)
    print(f"Utilization: {utilization:.1f}% | Free Blocks: {free_blocks} | "
          f"Active Processes: {len(allocations)}")

def allocate_memory(size=None):
    global pid_counter
    pid_counter += 1

    if mode == "paging":
        free_indices = np.where(memory == 0)[0]
        if len(free_indices) == 0:
            return False
        idx = np.random.choice(free_indices)
        memory[idx] = 1
        allocations.append((idx, 1, pid_counter))
        return True
    else:
        size = np.random.randint(2, 5) if size is None else size
        for i in range(16 - size + 1):
            if all(memory[i:i+size] == 0):
                memory[i:i+size] = 1
                allocations.append((i, size, pid_counter))
                return True
        return False

def allocate(btn):
    success = allocate_memory()
    update_display(f"Allocation failed - insufficient memory!" if not success else "")

def deallocate(btn):
    if allocations:
        start, size, pid = allocations.pop(0)
        memory[start:start+size] = 0
        update_display(f"Deallocated P{pid} (size: {size})")
    else:
        update_display("No allocations to deallocate!")

def toggle_mode(btn):
    global mode
    mode = "segmentation" if mode == "paging" else "paging"
    update_display(f"Switched to {mode} mode")

def reset_memory(btn):
    global memory, allocations, pid_counter
    memory = np.zeros(16)
    allocations = []
    pid_counter = 0
    update_display("Memory reset")

def update_display(message=""):
    clear_output(wait=True)
    plot_memory()
    if message:
        print(message)
    display(control_panel)

# Enhanced UI
alloc_button = widgets.Button(description="Allocate", button_style='success')
dealloc_button = widgets.Button(description="Deallocate", button_style='danger')
toggle_button = widgets.Button(description="Toggle Mode", button_style='info')
reset_button = widgets.Button(description="Reset", button_style='warning')

alloc_button.on_click(allocate)
dealloc_button.on_click(deallocate)
toggle_button.on_click(toggle_mode)
reset_button.on_click(reset_memory)

control_panel = widgets.HBox([alloc_button, dealloc_button, toggle_button, reset_button])
plot_memory()
display(control_panel)

