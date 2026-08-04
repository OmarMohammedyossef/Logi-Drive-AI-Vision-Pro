# GoogleTest Primer

This README contains an introduction  to the `gtest` 

## Installing gtest on Ubuntu
**Steps**
```bash
sudo apt-get install libgtest-dev
cd /usr/src/googletest/googletest
sudo mkdir build
cd build
sudo cmake ..
sudo make
sudo cp libgtest* /usr/lib/
cd ..
sudo rm -rf build
```

Then do this:
```bash
sudo mkdir /usr/local/lib/googletest
sudo ln -s /usr/lib/libgtest.a /usr/local/lib/googletest/libgtest.a
sudo ln -s /usr/lib/libgtest_main.a /usr/local/lib/googletest/libgtest_main.a
```

---

## Why GoogleTest?

GoogleTest is a C++ testing framework designed to help developers write better tests efficiently. It supports various test types, including unit tests, and works across multiple platforms (Linux, Windows, Mac).

### **Key Features:**

- **Independent & Repeatable Tests**
    
    - Ensures tests do not interfere with each other.
    - Runs each test on a separate object.
    - Allows isolated debugging of failing tests.
- **Organized & Maintainable Tests**
    
    - Groups related tests into test suites.
    - Encourages structured and easy-to-maintain test code.
- **Portable & Reusable**
    
    - Works across different OSes and compilers.
    - Supports configurations with or without exceptions.
- **Detailed Failure Reports**
    
    - Does not stop at the first failure. Instead, it only stops the current test and continues with the next.
    - Supports non-fatal failures to catch multiple issues in one run.
- **Automated Test Management**
    
    - Keeps track of all defined tests automatically.
    - No need to manually list tests to run them.
- **Optimized Performance**
    
    - Allows resource reuse across tests.
    - Reduces setup/teardown overhead while keeping tests independent.

---
## Resources
https://google.github.io/googletest/
