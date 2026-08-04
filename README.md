# Quantum-Circuit-Library
QuantumX school

"""
Quantum Circuit Library - Graphical User Interface (GUI)
--------------------------------------------------------
Developed by Yuhui (Mac Physics UTM)

A desktop application built using Tkinter, Qiskit, and Matplotlib to interactively
explore Superposition, Bell States, and GHZ States.
"""

import tkinter as tk
from tkinter import ttk, messagebox
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

from qiskit import QuantumCircuit
from qiskit.visualization import plot_histogram
from qiskit_aer import AerSimulator
from qiskit.quantum_info import Statevector

# Initialize AerSimulator backend
simulator = AerSimulator()

# --- Color Palette (Navy Theme) ---
BG_DARK = "#0a192f"       # Main background
BG_PANEL = "#112240"      # Cards/Panels
BG_CARD = "#1d2d50"       # Text areas/Inputs
TEXT_LIGHT = "#e6f1ff"    # Primary text
TEXT_MUTED = "#8892b0"    # Accent/Muted text
ACCENT_BLUE = "#64ffda"   # Highlights/Titles
LIGHT_BLUE = "#82b1ff"    # Count values text color & histogram bars
BTN_RED = "#ff5555"       # Exit button accent


# ==============================================================================
# REQUIRED REUSABLE FUNCTIONS (Assignment Specification)
# ==============================================================================

def create_superposition() -> QuantumCircuit:
    """Creates a 1-qubit circuit in an equal superposition state |+>."""
    qc = QuantumCircuit(1, 1)
    qc.h(0)
    return qc


def create_bell_state() -> QuantumCircuit:
    """Creates a 2-qubit maximally entangled Bell State (|Φ+>)."""
    qc = QuantumCircuit(2, 2)
    qc.h(0)
    qc.cx(0, 1)
    return qc


def create_ghz_state() -> QuantumCircuit:
    """Creates a 3-qubit maximally entangled GHZ State (|GHZ>)."""
    qc = QuantumCircuit(3, 3)
    qc.h(0)
    qc.cx(0, 1)
    qc.cx(1, 2)
    return qc


def show_statevector(circuit: QuantumCircuit) -> str:
    """Calculates and returns the Statevector representation before measurement."""
    sv = Statevector.from_instruction(circuit)
    return str(sv)


def run_circuit(circuit: QuantumCircuit, shots: int = 1024) -> dict:
    """Measures all qubits and executes the circuit using AerSimulator."""
    meas_circuit = circuit.copy()
    meas_circuit.measure_all(add_bits=False)
    result = simulator.run(meas_circuit, shots=shots).result()
    counts = result.get_counts()
    return counts


def show_histogram(counts: dict, title: str, parent_frame: ttk.LabelFrame) -> FigureCanvasTkAgg:
    """Renders the measurement histogram on a Matplotlib figure embedded in the GUI."""
    fig = plt.figure(figsize=(6, 5), facecolor=BG_PANEL)
    ax = fig.add_subplot(111)
    ax.set_facecolor(BG_PANEL)

    # Plot Qiskit Histogram with light blue bars
    plot_histogram(counts, ax=ax, title=title, color='#82b1ff')

    # Color customizations for Navy theme
    ax.title.set_color(TEXT_LIGHT)
    ax.xaxis.label.set_color(TEXT_LIGHT)
    ax.yaxis.label.set_color(TEXT_LIGHT)
    ax.tick_params(colors=TEXT_LIGHT)
    for spine in ax.spines.values():
        spine.set_color(BG_CARD)

    # Set light blue color for count values above bars
    for text in ax.texts:
        text.set_color(LIGHT_BLUE)

    fig.tight_layout()

    # Embed figure into Tkinter GUI
    canvas = FigureCanvasTkAgg(fig, master=parent_frame)
    canvas.draw()
    return canvas


# ==============================================================================
# GUI APPLICATION CLASS
# ==============================================================================

class QuantumLibraryApp:
    def __init__(self, root: tk.Tk):
        self.root = root
        self.root.title("Quantum Circuit Library")
        self.root.geometry("1100x760")
        self.root.minsize(900, 650)
        self.root.configure(bg=BG_DARK)

        # Style configuration for TTK Widgets
        self.style = ttk.Style()
        self.style.theme_use('clam')

        self.style.configure(".", background=BG_DARK, foreground=TEXT_LIGHT)
        self.style.configure("TLabelframe", background=BG_PANEL, foreground=ACCENT_BLUE, borderwidth=1, relief="solid")
        self.style.configure("TLabelframe.Label", background=BG_PANEL, foreground=ACCENT_BLUE, font=("Helvetica", 10, "bold"))
        self.style.configure("TFrame", background=BG_DARK)
        self.style.configure("Panel.TFrame", background=BG_PANEL)
        self.style.configure("TRadiobutton", background=BG_PANEL, foreground=TEXT_LIGHT, font=("Helvetica", 10))
        self.style.map("TRadiobutton", 
                       background=[('active', BG_PANEL)], 
                       foreground=[('active', ACCENT_BLUE)])

        self.explanations = {
            "Superposition": (
                "The Hadamard gate (H) puts the single qubit into an equal superposition state:\n"
                "|+⟩ = (|0⟩ + |1⟩) / √2.\n\n"
                "Upon measurement, the quantum state collapses, yielding approximately a 50% chance "
                "of measuring '0' and a 50% chance of measuring '1'."
            ),
            "Bell State": (
                "A Hadamard gate on q0 followed by a CNOT gate targeted at q1 puts the 2-qubit system "
                "into a maximally entangled Bell state: |Φ+⟩ = (|00⟩ + |11⟩) / √2.\n\n"
                "Due to quantum entanglement, measuring qubit 0 immediately determines the state of qubit 1. "
                "Hence, only |00⟩ and |11⟩ states appear with nearly equal probability."
            ),
            "GHZ State": (
                "Extending entanglement across 3 qubits via a Hadamard gate and two CNOT gates yields "
                "the Greenberger-Horne-Zeilinger (GHZ) state: |GHZ⟩ = (|000⟩ + |111⟩) / √2.\n\n"
                "All three qubits are intrinsically correlated; measuring one qubit collapses the entire "
                "system into either |000⟩ or |111⟩."
            )
        }

        self.canvas = None
        self._build_ui()

    def _build_ui(self):
        # Header Frame
        header_frame = tk.Frame(self.root, bg=BG_PANEL, padx=15, pady=10)
        header_frame.pack(fill=tk.X)

        title_lbl = tk.Label(
            header_frame, 
            text="⚛ Quantum Circuit Library", 
            font=("Helvetica", 18, "bold"),
            bg=BG_PANEL,
            fg=ACCENT_BLUE
        )
        title_lbl.pack(anchor=tk.W)

        # Developer Subtitle Credit
        subtitle_lbl = tk.Label(
            header_frame, 
            text="Quantum Circuit Library developed by Yuhui (Mac Physics UTM)", 
            font=("Helvetica", 10, "italic"),
            bg=BG_PANEL,
            fg=TEXT_MUTED
        )
        subtitle_lbl.pack(anchor=tk.W, pady=(2, 0))

        # Main Layout PanedWindow
        main_pane = ttk.PanedWindow(self.root, orient=tk.HORIZONTAL)
        main_pane.pack(fill=tk.BOTH, expand=True, padx=12, pady=12)

        # Left Frame: Control & Circuit Details
        left_frame = ttk.Frame(main_pane, style="Panel.TFrame", padding="10")
        main_pane.add(left_frame, weight=1)

        # Menu Selection
        select_group = ttk.LabelFrame(left_frame, text=" Select Circuit ", padding="10")
        select_group.pack(fill=tk.X, pady=(0, 10))

        self.circuit_var = tk.StringVar(value="Superposition")
        options = ["Superposition", "Bell State", "GHZ State"]
        for opt in options:
            rb = ttk.Radiobutton(
                select_group, 
                text=opt, 
                value=opt, 
                variable=self.circuit_var,
                command=self.execute_and_update
            )
            rb.pack(anchor=tk.W, pady=3)

        # Circuit Diagram Frame
        diagram_group = ttk.LabelFrame(left_frame, text=" Quantum Circuit Diagram ", padding="10")
        diagram_group.pack(fill=tk.X, pady=(0, 10))

        self.diagram_text = tk.Text(
            diagram_group, height=6, font=("Courier", 10), 
            bg=BG_CARD, fg=TEXT_LIGHT, insertbackground=TEXT_LIGHT, 
            relief="flat", highlightthickness=1, highlightbackground=BG_CARD
        )
        self.diagram_text.pack(fill=tk.X)

        # Statevector Frame
        sv_group = ttk.LabelFrame(left_frame, text=" Statevector (Before Measurement) ", padding="10")
        sv_group.pack(fill=tk.X, pady=(0, 10))

        self.sv_text = tk.Text(
            sv_group, height=4, font=("Courier", 10), 
            bg=BG_CARD, fg=TEXT_LIGHT, insertbackground=TEXT_LIGHT, 
            relief="flat", highlightthickness=1, highlightbackground=BG_CARD
        )
        self.sv_text.pack(fill=tk.X)

        # Explanation Frame
        exp_group = ttk.LabelFrame(left_frame, text=" Explanation ", padding="10")
        exp_group.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

        self.exp_text = tk.Text(
            exp_group, wrap=tk.WORD, font=("Helvetica", 10), 
            bg=BG_CARD, fg=TEXT_LIGHT, insertbackground=TEXT_LIGHT, 
            relief="flat", highlightthickness=1, highlightbackground=BG_CARD
        )
        self.exp_text.pack(fill=tk.BOTH, expand=True)

        # Control Buttons Frame (Restart & Exit)
        btn_frame = tk.Frame(left_frame, bg=BG_PANEL)
        btn_frame.pack(fill=tk.X)

        restart_btn = tk.Button(
            btn_frame, 
            text="🔄 Restart", 
            font=("Helvetica", 10, "bold"),
            bg=BG_CARD, 
            fg=ACCENT_BLUE,
            activebackground=BG_DARK,
            activeforeground=ACCENT_BLUE,
            relief="flat",
            cursor="hand2",
            command=self.restart_app
        )
        restart_btn.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=(0, 5))

        exit_btn = tk.Button(
            btn_frame, 
            text="❌ Exit", 
            font=("Helvetica", 10, "bold"),
            bg=BG_CARD, 
            fg=BTN_RED,
            activebackground=BG_DARK,
            activeforeground=BTN_RED,
            relief="flat",
            cursor="hand2",
            command=self.exit_app
        )
        exit_btn.pack(side=tk.RIGHT, fill=tk.X, expand=True, padx=(5, 0))

        # Right Frame: Measurement Histogram
        self.right_frame = ttk.LabelFrame(main_pane, text=" Measurement Histogram (1024 Shots) ", padding="10")
        main_pane.add(self.right_frame, weight=2)

        # Initial Run
        self.execute_and_update()

    def execute_and_update(self):
        selected = self.circuit_var.get()

        # 1. Generate Circuit
        if selected == "Superposition":
            qc = create_superposition()
        elif selected == "Bell State":
            qc = create_bell_state()
        elif selected == "GHZ State":
            qc = create_ghz_state()
        else:
            messagebox.showerror("Error", "Unknown circuit selection.")
            return

        # 2. Display ASCII Circuit Diagram
        self.diagram_text.delete("1.0", tk.END)
        self.diagram_text.insert(tk.END, str(qc.draw(output='text')))

        # 3. Calculate and Print Statevector before measurement
        sv_str = show_statevector(qc)
        self.sv_text.delete("1.0", tk.END)
        self.sv_text.insert(tk.END, sv_str)

        # 4. Print Explanation
        self.exp_text.delete("1.0", tk.END)
        self.exp_text.insert(tk.END, self.explanations[selected])

        # 5. Execute Circuit on AerSimulator
        counts = run_circuit(qc)

        # 6. Display Histogram
        if self.canvas:
            self.canvas.get_tk_widget().destroy()

        self.canvas = show_histogram(counts, f"Measurement Results: {selected}", self.right_frame)
        self.canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)

    def restart_app(self):
        """Resets the application selection to Superposition and refreshes."""
        self.circuit_var.set("Superposition")
        self.execute_and_update()

    def exit_app(self):
        """Closes the Tkinter GUI window cleanly."""
        self.root.destroy()


# --- Application Entry Point ---

def main():
    root = tk.Tk()
    app = QuantumLibraryApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
