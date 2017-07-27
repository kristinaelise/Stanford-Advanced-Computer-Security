# Buffer Overflows

---

# 6.1 Anatomy of a Buffer Overflow

**Buffer**: a particular amount of memory inside a program that may be used to store user input, or other form of untrusted data.
- typically have a fixed max size

**Buffer Overflow**: a buffer overflow occurs when the untrusted user input exceeds the max buffer size
- overflow data may end up in places you don't want it to 

## 6.1.1 A Small Example

`get_input` has a buffer of 1024 characters and makes a call to `get_string` to request user input

- if a user enters < 1024 characters, it'll all fit in `buf`
- what happens if the user enters more than 1024 characters of input?

## 6.1.2 A More Detailed Example

- let's say we have a program running on a PC and it's connected to a safe
- the goal of the program is to accept a password from the user, and only open the safe if the password is correct
- if I'm a malicious user, how can I get the program to open the safe without knowing the password? 

- main program says if `checkPassword` is true, call a function to open the vault
- expected functionality is if user enters right password, only in that case will it open

`checkPassword`
- buffer of 16 characters: `pass`
- zeros out the memory so that it's initialized
- prints out a prompt to the user: `Please enter password`
- calls `get_string` to have the user enter password
- compares string to `opensesame`
- if the password user enters is equal to `opensesame` will return 1; else 0

- stack data structure: `main` calls `checkPassword` and should return to `main`
  - computer reserves space to hold the password input
  - puts a return address on the stack so when it finishes executing `checkPassword`, it knows where to go next (the `main` function)
  - if input > 16 characters
    - first 16 characters would take up allocated space
    - each additional character will continue to be written to stack (as per `GetString`)
    - will continue being written to the stack and corrupt the stack
    - return address will get overwritten; if the attacker just inputs garbage the computer will try to jump to a return address specified by garbage and likely crash
  - if the attacker has some information about the program, can have the 17th - 20th character (since return addresses are 4 bytes in this example) specify a different return address that points to the open vault function

### `checkPassword()` Bugs
- attackers typically need a copy of the binary
  - if they can understand the binary structure they can figure out what the addresses are, and what to set them to
- the return address overwritten to be something already _within_ the program
  - you can also have attacks with `jump to code` instructions injected
- key problem is `get_s` will allow the user to type in an arbitrary amount of input

### Non-Executable Stacks Don't Solve it All    
- return address can be overwritten to point to code already in the program 

**Return-into-libc** attack: if you can execute a shell command, you can provide the OS whatever command you want
  
**Shell code**: code that will give you arbitrary control and let you run whatever command you want via a command shell

## 6.1.3 The `safe_gets()` function
- in addition to passing a buffer, pass a max number of characters the buffer can hold
  - conditions:
    - if input == null, then just return
    - if max char is 1, then populate the 0th element with null because can only accept empty string and return
    - otherwise, call `getchar` -- get one character at a time -- and put that in the buffer
      - continue filling buffer only until it's full (nb. last character has to be null)
- modify `checkPassword` to call this new `safe_gets()`

# 6.2 Safe String Libraries

**Don't use** `strcpy`, `strcat`, `sprintf`, `scanf` because they _don't check bounds._

Safer versions: `strncpy`, `strncat`, `fgets`.
- audit your code to make sure none of these unsafe functions are used
- but you need to make sure you pass the right buffer size to all of these functions otherwise you can still have a buffer overflow vulnerability
- typically only happens in programs that are fully compiled
  - interpreted programs have more checks in place
- in theory, Java compiler does enforce type safety, but possible there could be a buffer overflow in Java virtual machine

# 6.3 Additional Approaches

## 6.3.1 StackGuard
- invented by Cowan - a researcher
- compiler technique that makes the observation that if an attacker wants to take control of a running program it needs to overwrite return addr of some function
- can we determine - at runtime - whether return addr is being ovewritten?
- if so, can we stop the program? May result in DoS, but better than the alternative
- before a return address, you insert a **canary**

**Canary**: byte string that is known at run time
- canary defends the return address
- if someone tries to take control of program, they would have to overwrite canary first

- compile with an instrumented compiler that doesn't _just_ allocate stack frame and putting return address; it generates code that:
  - function is called, stack frame set up
  - space allocated for buffers, etc.
  - reserve space for the canary and put before return address
  - after function is finished but _before_ return address is jumped to check the value of the canary
    - is the value the same as what it was when populated? 
      - yes? great, continue
      - no? stop executing
- there are some ways to get around it -- interesting paper on SPD website and Foundations of Security website about defeating StackGuard
- most compilers offer canary protection

## 6.3.2 Static Analysis Tools

**Static Analysis**: analyzing programs without running them
- run a meta-level compiler over the code
- goal is to find security synchronization and memory bugs
- can notice that unsafe functions are being called, whether right buffer lenghts are being used, are buffers being checked before used?
- Coverity has an interesting paper on bugs in Linux device drivers

# 6.4 Performance
- very little performance cost
- performance shouldn't be the reason you _don't_ do checks in your programs

# 6.5 Heap-Based Overflows
- previous types were stack-based overflows
- having non-exe stacks doesn't really help because of return to lib c attacks and you can overflow buffers on the stack as _well_ as the heap (dynamically allocated memory)
  - ex. `malloc` or everytime you call `new`, you're allocating memory for a new object in the heap
- `malloc`, for example, requires a fixed sized memory
  - if attacker knows size of allocated fixed memory, can overrun buffers on the heap just as easily as on the stack
  - same kinds of fixes: bounds checking
- don't just exist for C and C++, but also in Javascript interpreters for spreading malware

# 6.6 Other Memory Corruption Vulnerabilities
- buffer overflows are a type of vulnerability due to _memory corruption_

## 6.6.1 Format String Vulnerabilities
- in C, `sprintf` writes data to a string with formatting
  - in this example, program allocates 10 chars for message and 8 chars for username
  - if attacker can populate username or message variables and provide larger input, `sprintf` will continue populating the buffer
  - may not be enough space, and can populate with shellcode or return address to generate a buffer overflow attack through a format string vulnerability 

## 6.6.2 Integer Overflows
- if programmers don't effectively check ranges on integers, attackers can change control flow of the program
- `int` expected to be between -2^32 and 2^32-1
- if you specify a value for the int that's larger, the value will wrap around and become negative
  - can cause programs to do unexpected things

### Example ###

`formatStr`: takes a buffer and buffer length
- function introduces some number of blank spaces before the name entered by a user
- offset refers to the number of those blank spaces
- `buflen` should be < `offset` + `slen` 
- message buffer has the output; takes message buffer to offset and populates with blank chars
  - copies string into the message buffer offset by a certain number of chars

_What happens if the attacker can specify an offset that's too large?_
- value of offset can become negative
- attacker will be able to write outside bounds of message buffer
- will be able to write into an area _before_ the message buffer

**Integer Overflow Vulnerability** - lets attackers write to places in memory that they're not supposed to.


