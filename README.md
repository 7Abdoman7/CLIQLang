# CLIQLang

CLIQLang is a quantum programming language for defining and executing quantum circuits using a custom syntax. It supports basic quantum gates, conditional operations, loops, and oracle gates.

## Requirements

- Linux x86_64 to run the AppImage.
- No Java required to run the AppImage.
- For building from source: Java 22+ and Maven.

## Execution

Run a CLIQ file using the AppImage:
   ```bash
   chmod +x CLIQLang-0.1.0-preview-x86_64.AppImage
   ./CLIQLang-0.1.0-preview-x86_64.AppImage <your-file.cliq>
   ```


Interactive Session Commands:
- `forward` (or `f`): Apply the next gate and show the new state.
- `backward` (or `b`): Revert the last gate and show the previous state.
- `reset` (or `r`): Reset the circuit to its initial |0...0> state.
- `run`: Run all gates from the current step to the end.
- `run <shots>`: Run the full circuit from the start for the given number of shots, measuring all qubits at the end of each shot and printing only a summary table of bitstring, count, and probability (no intermediate steps).
- `frun`: Run all gates to final state (no intermediate steps).
- `rrun`: Reset, then run all gates from the start.
- `exit` / `quit`: Exit the application.
- `help`: Show help message.
- `show <qubits>`: Show marginal probabilities over the selected qubits only (zero-probability outcomes omitted). Examples: `show 0`, `show 0,2`. Qubits are 0-based with the leftmost qubit as index 0.

## Noise & Fidelity

The simulator supports three configurable error models that can be set during interactive startup or programmatically:

- **Depolarization (probability p)**: After each gate, each affected target qubit receives a random Pauli from {X, Y, Z} with probability p.
- **Gate Fidelity (F ∈ [0,1])**: Models average gate fidelity. Internally approximated by a single-qubit Pauli channel with probability p = 2 · (1 − F) per affected qubit after each gate.
- **Decoherence (Amplitude Damping)**:
  - **Rate**: Probability per affected qubit after each gate that a damping event is attempted.
  - **Decay Probability**: If a damping event occurs, probability that |1⟩ decays to |0⟩. Implementation uses `QuantumDecoherence.decohere` and preserves normalization.

### Configure via Interactive Startup
When you launch the simulator without `--skip-config`, you’ll be prompted:
```
Noise Depolarization: 0.05
Decoherence Rate: 0.02
Decay Probability: 0.10
Gate Fidelity: 0.995
```

### Programmatic Usage (Java)
```java
CliqExecutor exec = new CliqExecutor(
    new File("circuit.cliq"),
    /* depolarizingProb */ 0.05,
    /* gateFidelity    */ 0.995,
    /* decoherenceRate */ 0.02,
    /* decayProbability*/ 0.10
);
```

## Syntax

### Quantum Gates

- **X, Y, Z, H, T, S**: Standard quantum gates applied to target qubits.
  
  ```
  X [control-qubits] [target-qubits]
  H [] [0]
  ```

- **Parametric Gates**: Various rotation gates with parameters.

  ```
  Rx(pi/4) [0] [1]
  Rz(0.24) [0,1] [2]
  Ph(pi/8) [] [3]
  RPh(pi/4) [1] [4]
  
  ```

- **SWAP Gate**: Exchange the states of two qubits.

  ```
  SWAP [] [0,1]
  ```

- **Range**: Apply operations over a range of qubits.

  ```
  X [] [range(0,3)]  # Apply X gate to qubits 0, 1, 2
  ```

### Measurement

- Measure specified qubits:

  ```
  ! [0,1,2]
  ```

### Conditional Operations

- Use `if` statements to perform operations based on measurement results:

  ```
  if [0] == 1:
      X [] [1]
  end
  ```

### Loops

- **For Loop**: Run a block of gates multiple times.

  ```
  for [3]:
      H [] [0]
  end
  ```

### Definitions (DEFINE)

Introduce file-scoped constants/macros that are substituted during parsing. Useful for angles, loop counts, ranges, and conditions.

**Syntax:**
```
DEFINE NAME = value
DEFINE "string key" = value   # legacy; prefer identifier form
```

- Values can be numbers or expressions (e.g., `pi/4`, `2*pi`).
- Substitutions are token-aware and apply to subsequent lines, including inside angles, ranges, and conditionals.

**Examples:**
```
# Angle constants
DEFINE THETA = pi/4
Rx(THETA) [] [0]
Rz(2*THETA) [] [1]

# Ranges and loop sizes
DEFINE N = 3
H [] [range(0,N)]

# Conditionals
DEFINE ONE = 1
if [0] == ONE:
    X [] [1]
end

# Macro-like alias (legacy quoted key)
DEFINE "HGate" = H
HGate [] [0]
```

### Macros (MACRO)

Define reusable, parameterized blocks of CLIQ code that expand at parse time.

**Syntax:**
```
MACRO Name [param1, param2, ...]:
    # body (can include gates, DEFINE usage, conditionals, loops)
end

# Invocation
Name [arg1, arg2, ...]
```

- Parameters are substituted token-aware throughout the macro body.
- Macro bodies can contain nested `if`, `for`, and `while` blocks; expansion respects nested `end` pairing.
- Macro names are identifiers; invocation uses the same name followed by `[args]` (no colon).

**Examples:**
```
DEFINE ANG = pi/4

MACRO RotateAndMeasure [angle, q]:
    Rz(angle) [] [q]
    ! [q]
end

RotateAndMeasure [ANG, 0]

# Parameterized utility
MACRO ControlH [ctrl, target]:
    if [ctrl] == 1:
        H [] [target]
    end
end

ControlH [0, 1]
```

### Custom Matrix Definitions

Define custom quantum gates using complex matrices. This allows you to create any quantum operation by specifying its matrix representation.

**Syntax:**
```
MATRIX: name = [matrix_elements]
```

**Examples:**
```
# Define a custom 2x2 gate (e.g., custom rotation)
MATRIX: MyGate = [0.707 0.707, -0.707 0.707]

# Use complex numbers with 'i' notation
MATRIX: PhaseGate = [1 0, 0 0.707+0.707i]

# Define a 4x4 gate for two-qubit operations
MATRIX: CustomCNOT = [1 0 0 0, 0 1 0 0, 0 0 0 1, 0 0 1 0]

# Apply the custom gate
MyGate [] [0]
PhaseGate [] [1]
```

**Matrix Format:**
- Elements separated by spaces
- Rows separated by commas
- Complex numbers: `real+imagi` or `real-imagi`
- Square matrices only (2x2, 4x4, 8x8, etc.)

### Oracle

An Oracle is a quantum operation that marks specific input states by flipping an output qubit. It's commonly used in quantum algorithms like Grover's search.

**Syntax:**
```
ORACLE [input-qubits] [output-qubit] [marked-states]
```

**Parameters:**
- `input-qubits`: The qubits that represent the input to the oracle
- `output-qubit`: The qubit that gets flipped when input matches a marked state
- `marked-states`: List of states to mark (can use decimal or binary format)

**Examples:**
```
# Mark states 0 and 3 using decimal format
ORACLE [0,1] [2] [0,3]

# Mark states using binary format (equivalent to above)
ORACLE [0,1] [2] b[00,11]

# Mark multiple states with range input
ORACLE [range(0,3)] [3] [1,5,7]

# Single input qubit oracle
ORACLE [0] [1] [1]
```

## Examples

- **Basic Gates**:
  ```
  H [] [0]
  X [0] [1]
  Y [0,1] [2]
  SWAP [] [2,3]
  ```

- **Rotation and Phase Gates**:
  ```
  Rx(pi/4) [0] [1]
  Rz(0.24) [0,1] [2]
  Ph(pi/8) [] [3]
  RPh(pi/4) [1] [4]
  ```