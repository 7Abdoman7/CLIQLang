# CLIQLang

CLIQLang is a quantum programming language for defining and executing quantum circuits using a custom syntax. It supports basic quantum gates, conditional operations, loops, and oracle gates.

## Requirements

- Java 22 or higher is required to run CLIQLang.

## Execution

Run a CLIQ file using the following command:
   ```bash
   java -jar CLIQLang_version.jar <your-file.cliq>
   ```


Interactive Session Commands:
- `forward` (or `f`): Apply the next gate and show the new state.
- `backward` (or `b`): Revert the last gate and show the previous state.
- `reset` (or `r`): Reset the circuit to its initial |0...0> state.
- `run`: Run all gates from the current step to the end.
- `exit` / `quit`: Exit the application.
- `help`: Show help message.

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

- **Loop Structure**:
  ```
  for [2]:
      X [] [0]
  end
  ```

- **Conditional Logic**:
  ```
  H [] [0]
  ! [0]
  if [0] == 1:
      X [] [1]
  end
  ```

- **Oracle Usage**:
  ```
  # Prepare superposition on input qubits
  H [] [0,1]
  
  # Apply oracle that marks states 01 and 10
  ORACLE [0,1] [2] b[01,10]
  
  # Measure all qubits
  ! [0,1,2]
  ```

- **Range Example**:
  ```
  H [] [range(0,3)]
  ```

- **Custom Matrix Example**:
  ```
  # Define a custom gate
  MATRIX: RotY45 = [0.9239 -0.3827, 0.3827 0.9239]
  
  # Use the custom gate
  RotY45 [] [0]
  ```

## Complete Example

Here's a complete example demonstrating multiple features:

```cliq
# Bell State Creation
H [] [0]          # Put qubit 0 in superposition
X [0] [1]         # Create entanglement with CNOT

# Measure and apply conditional operation
! [0]
if [0] == 1:
    Z [] [1]      # Apply Z gate if qubit 0 is measured as 1
end

# Loop example
for [3]:
    Rx(pi/6) [] [2]
end

# Final measurement
! [0,1,2]
```

## Output

The states of qubits and results of measurements are printed to the console as the circuit is executed, allowing for real-time tracking of the quantum state vectors. Each state is displayed with its amplitude and probability.

## Gate Reference

### Standard Gates
- `X`, `Y`, `Z`: Pauli gates
- `H`: Hadamard gate (creates superposition)
- `T`: T gate (π/8 phase gate)
- `S`: S gate (π/4 phase gate)
- `SWAP`: Swaps two qubit states

### Parametric Gates
- `Rx(angle)`: Rotation around X-axis
- `Ry(angle)`: Rotation around Y-axis  
- `Rz(angle)`: Rotation around Z-axis
- `Ph(angle)`: Global phase gate
- `RPh(angle)`: Relative phase gate

### Angle Expressions
Angles can be expressed as:
- `pi/4`, `pi/2`, `2*pi`
- Decimal values: `0.785`, `1.57`
- Negative values: `-pi/2`

### Range Function
- `range(n)`: Qubits 0 to n-1
- `range(start,end)`: Qubits from start to end-1
- Example: `range(1,4)` = qubits 1, 2, 3

## License

This project is licensed under the MIT License - see the LICENSE file for details.
