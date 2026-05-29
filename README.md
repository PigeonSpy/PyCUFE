# PyCUFE
A Pythonic-Sage implementation of the Ciphertext Updatable Functional Encryption (CUFE) scheme based on works of Valerio Cini et al

# Installation
## X. Prerequisites
You must install the Windows Subsystem from Linux (WSL) environment to run the code with code developed in Python 3.12.12 via VSCode.

## 1. Installing WSL
Instructions vary depending on the IDE you use and Windows version. We provide the link to the VSCode installation:
https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-vscode.

## 2. Installing Sage
The code requires Sage to run. Sage is a library providing the necessary operations for lattice and matrix manipulation which is required for implementation of the Learning With Errors approach used to construct the scheme. You can refer to the following for SageMath installation (Please choose the "Windows" option if using WSL, I have not tested any other installations!): https://doc.sagemath.org/html/en/installation/index.html.

## 3. Setting up the project
Make sure to install the Python extensions (if using VSCode) and the Jupyter extensions (to run the notebook code!)

If on VSCode follow the steps in this guide to start a WSL environment (https://code.visualstudio.com/docs/remote/wsl). Once in this environment, clone the project. Make sure you install the Python extensions for VSCode and install Python 3.12.12 on your WSL system then start a virtual environment (Ctrl+Shift+P to open Command Palette and search Select Environment) clicking this you should see a list of environments, you want to select the one titled: "Python 3.12.12 (Sage)", if you don't see it then refer to installing Sage again. You should now be able to run the code.

# Remarks
The code is somewhat incomplete, while the functions work as intended, error values can accumulate for certain parameter choices and drown out the message signal returning an erroneous output. This is partly due to parameters not having their bounds enforced correctly allowing for incorrect parameter combinations and an issue on my part. My intention is to fix these. This code is done for my dissertation in general, demonstrating a working "toy" construction of the scheme without worrying about a secure fully-trusted authority.

# URL references
https://eprint.iacr.org/2022/1284.pdf - (Inner Product) Functional Encryption with Updatable Ciphertexts (For the main CUFE construction)
https://link.springer.com/chapter/10.1007/978-3-642-13190-5_28#preview - Efficient Lattice (H)IBE in the Standard Model (For the sampleLeft algorithm)
https://advancedcrypto.github.io/Lecture4a.pdf - Fully Homomorphic Encryption (Lecture notes) (For the gadget matrix)
https://people.csail.mit.edu/vinodv/6876-Fall2015/L16.pdf - Sampling Lattice Trapdoors (Lecture notes) (For trapdoor sampling)

# Acknowledgments
Thank you to my supervisor for the supporting me with the project and project direction. Additional thanks to the papers outlined for providing the basis for construction.
