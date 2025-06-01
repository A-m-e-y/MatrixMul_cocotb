# MatrixMul_cocotb
This project aims to build a verilog module for Matrix Multiplication and test it via cocotb

This project implements a **hardware/software co-simulation framework** for matrix multiplication using a custom Verilog module and the [cocotb](https://www.cocotb.org/) Python-based verification framework. The goal is to verify and test a hardware matrix multiplication engine using Python, numpy, and cocotb, enabling easy comparison between software and hardware results.

---

## Project Overview

- **Hardware:** A Verilog module (`MatrixMulEngine.v`) that performs matrix multiplication on floating-point matrices.
- **Testbench:** A cocotb-based Python testbench (`test_matrix_mul.py`) that drives the hardware, loads matrices, and collects results.
- **Software:** Python scripts to generate random matrices, perform software matrix multiplication (using numpy), and compare results with hardware.
- **Automation:** A Makefile to run the cocotb simulation and orchestrate the workflow.
- **Data Exchange:** Input and output buffers (`input_buffer.txt`, `output_buffer.txt`) are used to pass data between Python and the hardware simulation.

---

## Cocotb Framework Explanation

**cocotb** is a coroutine-based cosimulation library for writing VHDL and Verilog testbenches in Python. In this project, cocotb is used to:

- Drive the Verilog hardware simulation from Python.
- Load input matrices into the hardware.
- Trigger the computation.
- Read back the results.
- Compare hardware results with software (numpy) results.

The workflow is as follows:

1. Python generates random matrices and writes them to `input_buffer.txt`.
2. The cocotb testbench (`test_matrix_mul.py`) reads this file, loads the matrices into the Verilog DUT, and starts the computation.
3. When the hardware signals completion, cocotb reads the output matrix and writes it to `output_buffer.txt`.
4. Python reads this file, reconstructs the matrix, and compares it to the numpy result.

---

## File-by-File Explanation

### 1. `RTL/MatrixMulEngine.v`
- **Role:** The Verilog module implementing the matrix multiplication engine. It accepts matrices A and B, computes C = A × B, and signals when done.
- **Interface:** Exposes ports for matrix data, dimensions, control signals (`start`, `done`), and clock/reset.

### 3. `Makefile`
- **Role:** Automates the cocotb simulation.
- **Key Variables:**
  - `VERILOG_SOURCES`: Points to the Verilog source file.
  - `TOPLEVEL`: Name of the top-level Verilog module.
  - `MODULE`: Name of the Python testbench module (`test_matrix_mul`).
  - `SIM`: Specifies the simulator (e.g., `icarus` for Icarus Verilog).
- **How it works:** When you run `make`, it launches the simulator, loads the Verilog module, and runs the cocotb Python testbench.

### 4. `matrix_hw_wrapper.py`
- **Role:** Python wrapper to interface with the hardware simulation.
- **Functionality:**
  - Accepts numpy matrices A and B.
  - Writes their dimensions and data to `input_buffer.txt`.
  - Runs the cocotb simulation via `make`.
  - Waits for `output_buffer.txt` to appear.
  - Reads the output matrix C from the file and returns it as a numpy array.
- **Purpose:** Provides a simple function (`matrix_mul_hw`) to perform hardware matrix multiplication from Python.

### 5. `do_matrix_mul.py`
- **Role:** Main Python script to demonstrate the full workflow.
- **Functionality:**
  - Generates random matrices A and B.
  - Computes C_sw = A × B using numpy (software).
  - Computes C_hw = A × B using the hardware (via `matrix_hw_wrapper.py` and cocotb).
  - Prints both results and compares them.
- **Purpose:** Entry point for users to test the hardware/software co-simulation.

### 6. `test_matrix_mul.py`
- **Role:** The cocotb Python testbench.
- **Functionality:**
  - Reads matrix dimensions and data from `input_buffer.txt`.
  - Drives the Verilog DUT: loads matrices, sets control signals, and waits for completion.
  - Reads the output matrix from the DUT.
  - Writes the result to `output_buffer.txt`.
- **Purpose:** Bridges the Python and Verilog worlds, enabling automated, Python-driven hardware testing.

### 7. `input_buffer.txt` / `output_buffer.txt`
- **Role:** Temporary files for data exchange between Python and cocotb/Verilog.
- **`input_buffer.txt`:** Written by Python, read by cocotb. Contains matrix dimensions and data.
- **`output_buffer.txt`:** Written by cocotb, read by Python. Contains the result matrix.

---

## Typical Workflow

1. **Run the main script:**
   ```sh
   python do_matrix_mul.py
   ```
2. **What happens:**
   - Random matrices are generated.
   - Software multiplication is performed and printed.
   - Hardware multiplication is triggered (via cocotb and Verilog simulation).
   - Hardware result is printed.
   - Results are compared and differences (if any) are reported.

---

## Example Output

```
❯ python3 do_matrix_mul.py
Generating random matrices: A(2,4), B(4,2)
Running software matrix multiplication...

C_sw (shape (2, 2)):
    0.0638   -0.6152
    0.5099    0.7752
Running hardware matrix multiplication via cocotb...
rm -f results.xml
"make" -f Makefile results.xml
make[1]: Entering directory '/mnt/d/PSU/HW_For_AI_teuscher/MatrixMul_cocotb'
rm -f results.xml
MODULE=test_matrix_mul TESTCASE= TOPLEVEL=MatrixMulEngine TOPLEVEL_LANG=verilog \
         /usr/bin/vvp -M /usr/local/lib/python3.12/dist-packages/cocotb/libs -m libcocotbvpi_icarus   sim_build/sim.vvp
     -.--ns INFO     gpi                                ..mbed/gpi_embed.cpp:79   in set_program_name_in_venv        Did not detect Python virtual environment. Using system-wide Python interpreter
     -.--ns INFO     gpi                                ../gpi/GpiCommon.cpp:101  in gpi_print_registered_impl       VPI registered
     0.00ns INFO     cocotb                             Running on Icarus Verilog version 12.0 (stable)
     0.00ns INFO     cocotb                             Running tests with cocotb v1.9.2 from /usr/local/lib/python3.12/dist-packages/cocotb
     0.00ns INFO     cocotb                             Seeding Python random module with 1748648212
     0.00ns INFO     cocotb.regression                  Found test test_matrix_mul.matrix_mul_test
     0.00ns INFO     cocotb.regression                  running matrix_mul_test (1/1)
     0.00ns INFO     cocotb                             Starting matrix_mul_test...
   540.00ns INFO     cocotb                             Matrix multiplication complete. Results written to output_buffer.txt.
   540.00ns INFO     cocotb.regression                  matrix_mul_test passed
   540.00ns INFO     cocotb.regression                  *****************************************************************************************
                                                        ** TEST                             STATUS  SIM TIME (ns)  REAL TIME (s)  RATIO (ns/s) **
                                                        *****************************************************************************************
                                                        ** test_matrix_mul.matrix_mul_test   PASS         540.00           1.65        328.15  **
                                                        *****************************************************************************************
                                                        ** TESTS=1 PASS=1 FAIL=0 SKIP=0                   540.00           2.10        257.57  **
                                                        *****************************************************************************************

make[1]: Leaving directory '/mnt/d/PSU/HW_For_AI_teuscher/MatrixMul_cocotb'

C_hw (shape (2, 2)):
    0.0638   -0.6152
    0.5099    0.7752
✅ HW and SW results match within tolerance.

```

---

## Summary Table

| File                    | Description                                                                                 |
|-------------------------|---------------------------------------------------------------------------------------------|
| `RTL/MatrixMulEngine.v` | Verilog hardware module for matrix multiplication                                           |
| `RTL/tb_MatrixMulEngine.v` | Traditional Verilog testbench (not used by cocotb, but useful for debugging)             |
| `Makefile`              | Automates cocotb simulation and hardware/software co-simulation                            |
| `matrix_hw_wrapper.py`  | Python wrapper to run hardware matrix multiplication and handle data exchange               |
| `do_matrix_mul.py`      | Main Python script: generates matrices, runs SW/HW multiplication, compares results         |
| `test_matrix_mul.py`    | cocotb Python testbench: loads matrices, drives DUT, writes results                        |
| `input_buffer.txt`      | Temporary file: Python → cocotb/Verilog (matrix data and dimensions)                        |
| `output_buffer.txt`     | Temporary file: cocotb/Verilog → Python (result matrix)                                     |

---

## How to Extend

- **Change matrix sizes:** Edit `do_matrix_mul.py` to set different values for M, K, N.
- **Test with custom matrices:** Modify `generate_random_matrices` or directly assign values.
- **Debug hardware:** Use `RTL/tb_MatrixMulEngine.v` for standalone Verilog simulation.
- **Improve accuracy:** Adjust the tolerance in `compare_results`.

---

## Requirements

- Python 3.x
- numpy
- cocotb
- Icarus Verilog (or another supported simulator)

---

## Conclusion

This project demonstrates a robust, Python-driven verification flow for hardware matrix multiplication using cocotb. It enables rapid prototyping, easy debugging, and direct comparison between hardware and software results, all orchestrated from Python.

---