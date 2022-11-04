# Before we start

## Prerequisites

[Install Mars](https://courses.missouristate.edu/kenvollmar/mars/download.htm)

## Learning materials

[Tutorial](https://www.youtube.com/playlist?list=PL5b07qlmA3P6zUdDf-o97ddfpvPFuNa5A)

[Registers](https://en.wikibooks.org/wiki/MIPS_Assembly/Register_File)

[Instructions](https://jarrettbillingsley.github.io/teaching/classes/cs0447/guides/instructions.html)

[Syscall](https://courses.missouristate.edu/kenvollmar/mars/help/syscallhelp.html)

## Cheat sheet

### Print string, char, int, float, double

```asm
.data
	myString: .asciiz "Hello, I'm Hieu\n"
	myChar: .byte 'H'
	myInt: .word 20
	myFloat: .float 3.14
	myDouble: .double 20.09

.text
main:
	# Print string
	li $v0, 4           # load immediate
	la $a0, myString    # load address
	syscall

	# Print character
	li $v0, 4
	la $a0, myChar
	syscall

	# Print integer
	li $v0, 1
	lw $a0, myInt       # load word
	syscall	

	# coprocessor 1 is the place for floats, doubles

	# Print float
	li $v0, 2
	lwc1 $f12, myFloat     # lwc1 - load word to coproc 1
	syscall

	# Print double
	li $v0, 3
	ldc1 $f12, myDouble    # ldc1 - load double to coproc 1
	syscall

	# End of the program
	li $v0, 10
	syscall
```

### Arithmetic Instructions

```asm
.data
	myString: .asciiz "Hello, I'm Hieu\n"
	myChar: .byte 'H'
	myInt: .word 20
	myFloat: .float 3.14
	myDouble: .double 20.09

.text
main:
	# Print string
	li $v0, 4           # load immediate
	la $a0, myString    # load address
	syscall

	# Print character
	li $v0, 4
	la $a0, myChar
	syscall

	# Print integer
	li $v0, 1
	lw $a0, myInt       # load word
	syscall	

	# coprocessor 1 is the place for floats, doubles

	# Print float
	li $v0, 2
	lwc1 $f12, myFloat     # lwc1 - load word to coproc 1
	syscall

	# Print double
	li $v0, 3
	ldc1 $f12, myDouble    # ldc1 - load double to coproc 1
	syscall

	# End of the program
	li $v0, 10
	syscall
```

### Stack pointer `$sp` &  Using labels

```asm
.data

.text
main:
	li $s0, 10         # saved register $s is used with stack pointer
	jal printDoubleValue    # jump and link to a label

	# End of the program
	li $v0, 10
	syscall
	# This exit syscall is important
	# if you don't exist, the code will continue running
	# then hit the 'jr $ra', get back to the 'main' label
	# and create an infinitive loop


printDoubleValue:
	# Allocate
	addi $sp, $sp, -4  # gain 4 bytes from the stack for an int
	sw $s0, 0($sp)     # store word $s0 with the offset 0
	
	add $s0, $s0, $s0  # s0 = s0 x 2 = 20
	
	# Print the x2 value
	li $v0, 1
	move $a0, $s0      # copy the value from $s0 to $a0
	syscall

	# Get the previous state of $s0
	lw $s0, 0($sp)
	addi $sp, $sp, 4

	# Get back to the previous label
  # aka the one which called this label
	jr $ra    # jump return - return address
```

### Arguments, return values and nested functions

```asm
.data

.text
main:
	li $s0, 10

	jal dummyLabel    # $ra = main label

	# End of the program
	li $v0, 10
	syscall

addInt:
	# By convention, $v1 is the return value
	add $v1, $a1, $a2 # arguments
	jr $ra            # jump back to the $ra = dummyLabel label - aka 'jal addInt'

dummyLabel:
	addi $sp, $sp, -8
	sw   $s0, 0($sp)
	sw   $ra, 4($sp)

	# We must store the $ra of 'main' label to the stack
	# so that we can restore the $ra to 'main' later

	addi $s0, $s0, 20  # s0 = 10 + 20 = 30
	move $a1, $s0      # a1 = s0 = 30
	li   $a2, 100
	jal  addInt        # v1 = a1 + a2 = 30 + 100 = 130, $ra = dummyLabel label

	# Print the return value $v1
	li $v0, 1
	move $a0, $v1
	syscall

	lw $s0, 0($sp)     # s0 = 10 (restore the previous value)
	lw $ra, 4($sp)     # $ra = main label
	addi $sp, $sp, 8

	jr $ra             # jump back to the $ra = main label - aka 'jal dummyLabel'
```

### Get user input

```asm
.data
	promptName: .asciiz "Your name: "
	promptAge: .asciiz "Your age: "
	newLine: .asciiz "\n"
	name: .space 20    # name is a 20-byte string

.text
main:
	# Prompt name
	li $v0, 4
	la $a0, promptName
	syscall

	# Get name
	li $v0, 8
	la $a0, name  # data type
	li $a1, 20    # data size
	# this syscall requires v0, a0, a1
	syscall

	# Prompt age
	li $v0, 4
	la $a0, promptAge
	syscall

	# Get age
	li $v0, 5
	syscall
	move $t0, $v0  # store age in $t0

	# Print the user info
	li $v0, 4
	la $a0, newLine
	syscall

	li $v0, 4
	la $a0, name
	syscall

	li $v0, 1
	move $a0, $t0
	syscall

	# End of the program
	li $v0, 10
	syscall
```

### Array, While loop, Conditionals

```asm
.data
	myArray: .space 12  # array of 3 integers
	newLine: .asciiz "\n"

.text
main:
	li $t0, 0  # $t0 is our array indexing
	li $s0, 69
	li $s1, 96
	li $s2, 100
	
	# Update the elements
	sw $s0, myArray($t0)  # myArray[0]
	addi $t0, $t0, 4
	sw $s1, myArray($t0)  # myArray[1]
	addi $t0, $t0, 4
	sw $s2, myArray($t0)  # myArray[2]

	# Use While loop to print the array
	li $t0, 0  # set the index to 0

	while:  # This is a normal label named "while"
		beq $t0, 12, exit     # if $t0 = 12, branch to exit
		
		lw $t1, myArray($t0)  # temp

		# Print the element
		li $v0, 1
		move $a0, $t1
		syscall
		
		# Print new line
		li $v0, 4
		la $a0, newLine
		syscall

		addi $t0, $t0, 4      # update the index

		j while               # jump back to the while label
	exit:   # This is a normal label named "exit"

	# End of the program
	li $v0, 10
	syscall
```

### Array Initializer

```asm
.data
	myArray: .word 69:3  # array of 3 integers with the value of 69
	myArray2: .word 69 96 100  # array of 3 integers with different values
	newLine: .asciiz "\n"

.text
main:
	# Use While loop to print the array
	li $t0, 0  # set the index to 0

	while:  # This is a normal label named "while"
		beq $t0, 12, exit     # if $t0 = 12, branch to exit
		
		lw $t1, myArray2($t0)  # temp

		# Print the element
		li $v0, 1
		move $a0, $t1
		syscall
		
		# Print new line
		li $v0, 4
		la $a0, newLine
		syscall

		addi $t0, $t0, 4      # update the index

		j while               # jump back to the while label
	exit:   # This is a normal label named "exit"

	# End of the program
	li $v0, 10
	syscall
```

### Floating Point Arithmetic

```asm
.data
	float1: .float 3.14
	float2: .float 17.4
	double1: .double 20.09
	double2: .double 31.05

.text
main:
	# You should use the even-numbered registers
	# to store these values, idk why actually :)
	# Especially the doubles, must be even-numbered

	lwc1 $f2, float1
	lwc1 $f4, float2

	ldc1 $f6, double1
	ldc1 $f8, double2

	li $v0, 2
	add.s $f12, $f2, $f4
	syscall

	li $v0, 3
	add.d $f12, $f6, $f8
	syscall

	# The other instructions is
	# sub.s, sub.d, mul.s, mul.d, div.s, div.d

	# End of the program
	li $v0, 10
	syscall
```

### If statements using with floats and doubles

| **Instruction** | **Descripntion**               |
| --------------- | ------------------------------ |
| `c.eq.d`        | Compare equal double precision |
| `c.eq.s`        | Compare equal single precision |
| `c.le.d`        | Compare less or equal double precision |
| `c.le.s`        | Compare less or equal single precision |
| `c.lt.d`        | Compare less than double precision |
| `c.lt.s`        | Compare less than single precision |

```asm
# instruction reg1, reg2
c.eq.d $f0, $f2
```

To use the result of the condition, use `bc1t` if true or `bc1f` if false.

```asm
# bc1t/bc1f label
bc1t exit  # branch to exit label
```
