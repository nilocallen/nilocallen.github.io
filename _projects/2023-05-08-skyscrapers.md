---
layout: project
title:  Skyscrapers Assembly Backtracker
date:   2023-05-08 14:30:55 +0300
image:  /assets/images/projects/skyscrapers/cover.png
author: Colin Allen
tags:   Programming Project
---

A MIPS assembly program to solve Skyscraper puzzles using recursive backtracking.

I've got a fun one here!  This project was my first use of assembly and sparked my interest in low level interactions.  This program reads a specially formatted file to create the puzzle shown in the thumbnail, then solves it using recursive backtracking.  Because assembly code is hard to read and this project is nearly 1700 lines of it, I won't show code examples, and I'll instead include all of the code at the end.<br><br><br><br>

<h4>What is the Skyscrapers puzzle?</h4>
Skyscrapers puzzles involve a square grid surrounded by hints, and optionally including fixed buildings.  
<div style="text-align: center;">
  <img src="/assets/images/projects/skyscrapers/unsolved-solved.png" alt="Skyscrapers puzzles">
</div>
Puzzles are solved by placing down every number from one to n in each row and column, where n is the size of the board.  Each number corresponds to a skyscraper, where the number represents that skyscraper's height.  Ones represent the shortest skyscraper, and n represents the tallest. 

Each hint corresponds to the number of skyscrapers that can be seen if you were looking down that row or column.  Shorter skyscrapers behind taller ones cannot be seen.  Here's some examples, taken from <a href="https://www.brainbashers.com/skyscrapershelp.asp">BrainBashers</a>:
<div style="text-align: center;">
  <img src="/assets/images/projects/skyscrapers/examples.png" alt="Skyscrapers examples">
</div>

A puzzle is solved when the grid is filled, the numbers 1-n appear in every row and column, and the hints properly reflect how many skyscrapers can be seen from each row and column.  

<h4>What is the best way for a computer to approach this problem?</h4>
This can be solved in a few ways, but the simplest elegant solution is with <a href="https://brilliant.org/wiki/recursive-backtracking/#:~:text=Backtracking%20can%20be%20thought%20of,(one%20of%20the%20leaves).">recursive backtracking</a>.  Backtracking entails exploring promising solutions until there are no good ways to progress, then "backtracking" the decision that got you there and exploring a new one.  It is helpful for situations where exploring every state is not feasible because of quantity, and is especially easy to implement if there are rigid rules, like the ones described above, that clearly indicate a good versus bad state.  


<h4>How does this program work?</h4>
The program begins by first reading in a text file with one number per line that is structured as follows:
<div style="text-align: center;">
	board size, n<br>
	hint<sub>0</sub><br>
	hint<sub>1</sub><br>
	...<br>
	hint<sub>n<sup>2</sup></sub><br>
	number of fixed buildings, N<br>
	x<sub>0</sub><br>
	y<sub>0</sub><br>
	building<sub>0</sub><br>
	...<br>
	x<sub>N</sub><br>
	y<sub>N</sub><br>
	building<sub>N</sub>
</div>
Then, it recursively backtracks to solve the puzzle.  
<h5>How does recursion work in assembly?</h5>
This was the most interesting part of the project for me, as it highlights the power of the <a href="https://en.wikipedia.org/wiki/Stack_(abstract_data_type)">stack</a>.  A stack is an abstract data structure that stores elements similar to a stack of books in the real world; the items added first sit on the bottom of the stack, and new items are placed on top of it.  Items can only be taken from the stack from the top.  In order to explore different board states but also be able to revert back to previously explored states when needed, this program allocates all of its variables onto a stack.  

A stack frame is our standardized unit that is stored on the stack.  It is just a block of memory big enough to hold the board state.  Whenever we explore a new board state, we add it to the stack.  Then, when we wish to backtrack to a better state, we simply remove from the stack.  This is implemented using <a href="https://en.wikipedia.org/wiki/Recursion_(computer_science)">recursion</a>, a common technique in computer science that allows us to call the same function repeatedly to achieve this traversing of the stack.  

<h4>Here's the code.  Thanks for the read!</h4>
<pre>
	<code>
# File:		skyscrapers.asm
# Author:	Colin Allen
#
# Description:	Skyscrapers
#	This program takes input to generate
#	a skyscraper puzzle and then solves it.
#

#
# Name:		Constant definitions
#
# Description:	These are the constants used for
#	system calls and other needed values.
#

# Constants for system calls
#
PRINT_INT	= 1	# code for syscall to print int
PRINT_STRING	= 4	# code for syscall to print str
READ_INT	= 5	# code for syscall to read int

#
# Name:		Data areas
#
# Description:	Much of the data for the program is here
#

#
# The strings for the welcome msgs
#
	.data
	.align 0

str_init_top:	.asciiz "*******************\n"
str_init_mid:	.asciiz "**  SKYSCRAPERS  **\n"
#
# The strings for printing the board states
#
str_initial:	.asciiz "Initial Puzzle\n\n"
str_final:	.asciiz "Final Puzzle\n\n"
#

#
# The strings for printing the board
#
str_line:	.asciiz "---"
str_plus:	.asciiz "+"
str_vertical:	.asciiz "|"
#

#
# The strings for printing errors
#
str_err1:	.asciiz "Invalid board size, Skyscrapers terminating\n"
str_err2:	.asciiz "Illegal input value, Skyscrapers terminating\n"
str_err3:	.asciiz "Invalid number of fixed values, Skyscrapers terminating\n"
str_err4:	.asciiz "Illegal fixed input values, Skyscrapers terminating\n"
str_err5:	.asciiz "Impossible Puzzle\n"
#

#
# Miscellaneous useful strings.
#

newline:	.asciiz "\n"
str_4space:	.asciiz "    "
str_3space:	.asciiz "   "
str_2space:	.asciiz "  "
str_space:	.asciiz " "

#
# Arrays that will be passed as parameters for the problems.
#
	.align	2
hints:		.space 140	# room for 35 hints, 3 more than max
buildings:	.space 784	# room for 64 buildings, little more than max

#
# Problem-specific info that will be helpful later
#
board_size:		.space 4
num_fixed_buildings:	.space 4

#
# Name: Main program
#
# Description:	Driver for the program.  
#
	.text
	.align	2	# code must be on word boundaries
	.globl 	main	# main is a global label
	.globl 	solve_skyscrapers	# external function call
	.globl 	hints
	.globl 	buildings
	.globl 	board_size
	.globl 	num_fixed_buildings
	.globl 	print_arr
main:
M_FRAMESIZE = 8
	addi	$sp, $sp, -M_FRAMESIZE	# allocate space for the return address
	sw	$ra, -4+M_FRAMESIZE($sp)# store the ra on the stack

	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

	jal	print_banner

	jal	read_input
	move	$s0, $v0		# $s0 contains n
	move	$s1, $v1		# $s1 contains num fixed buildings

	la	$t0, board_size
	sw	$s0, 0($t0)
	la	$t0, num_fixed_buildings
	sw	$s1, 0($t0)

	li	$v0, PRINT_STRING
	la	$a0, str_initial
	syscall

	move	$a0, $s0
	move	$a1, $s1
	jal	print_unsolved

	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

	move	$a0, $s0
	move	$a1, $s1
	jal	solve_skyscrapers
	beq	$v0, $zero, throw_error_5
	

	la	$a0, str_final
	li	$v0, PRINT_STRING
	syscall


	move	$a0, $s0
	la	$a1, num_fixed_buildings
	lw	$a1, 0($a1)
	jal	print_unsolved

	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

#
# Finished!
#
main_done:
	lw	$ra, -4+M_FRAMESIZE($sp)	# restore the ra
	addi	$sp, $sp, M_FRAMESIZE	# deallocate stack space
	jr	$ra			# return from main

#
# Name:		throw_error_5
# Description: 	Throws an error if the puzzle is unsolvable
#
throw_error_5:
	la	$a0, str_err5
	li	$v0, PRINT_STRING
	syscall
	la	$a0, newline
	syscall
	j	main_done

#
# Name:		read_input
#
# Description:	Reads the input from the user and stores the values of the 
#		corresponding problem in appropriate registers.
# Returns:	$v0: board dimensions $v1: number of fixed buildings
#

read_input:
	addi	$sp, $sp, -M_FRAMESIZE
	sw	$ra, -4+M_FRAMESIZE($sp)

	li	$v0, READ_INT
	syscall
	move	$s0, $v0	# $s0 contains board dimensions
	mul	$s1, $s0, 4	# $s1 contains amount of times to read board sides
	
#
# Error checking if board size valid
#
	
	slti	$t0, $s0, 9
	la	$a0, str_err1
	beq	$t0, $zero, print_err	# if board too big, error
	slti	$t0, $s0, 3
	bne	$t0, $zero, print_err	# if board too small, error

#
# Board dimensions in valid range; loop through getting hints
#
	
	la	$s6, hints	# $s6 contains base address of hint array
	move	$s3, $zero	# $s3 contains counter for looping hints

hint_loop:
	beq	$s3, $s1, hint_loop_done
	li	$v0, READ_INT
	syscall
	move	$t1, $v0	# $t1 hold the next int read

#
# Error checking each input value between 0...n
#
	addi	$t9, $s0, 1
	slt	$t0, $t1, $t9
	la	$a0, str_err2
	beq	$t0, $zero, print_err

#
# Number valid; store in hints array
#
	
	sw	$t1, 0($s6)
	addi	$s6, $s6, 4
	addi	$s3, $s3, 1
	j	hint_loop

hint_loop_done:

#
# Get number of fixed buildings and error check (n > 0)
#
	li	$v0, READ_INT
	syscall
	move	$t1, $v0	# $t1 holds number of fixed buildings
	slt	$t0, $t1, $zero
	la	$a0, str_err3
	bne	$t0, $zero, print_err
	move	$s5, $t1	# $s5 holds valid number of fixed buildings
	mul	$s1, $t1, 3

	la	$s7, buildings	# $s7 contains base address of buildings array
#
# Number valid; loop through building data
#

buildings_loop:
	beq	$s1, $zero, buildings_loop_done
	li	$v0, READ_INT
	syscall
	move	$t9, $v0	# t9 holds first val read
	move	$a0, $t9
	jal	check_building_rows_cols
#
# First val valid, store in buildings array
#
	sw	$t9, 0($s7)
	addi	$s7, $s7, 4
#
# Get second val
#
	li	$v0, READ_INT
	syscall
	move	$t9, $v0	# $t9 holds second val in triplet
	move	$a0, $t9
	jal	check_building_rows_cols
#
# Second val valid, store in buildings array
#
	sw	$t9, 0($s7)
	addi	$s7, $s7, 4
#
# Get third val
#
	li	$v0, READ_INT
	syscall
	move	$t9, $v0	# $t9 has third val in triplet
#
# Error check third val
#
	li	$t3, 1
	slt	$t0, $t9, $t3
	la	$a0, str_err4
	bne	$t0, $zero, print_err

	addi	$t3, $s0, 1
	slt	$t0, $t9, $t3
	la	$a0, str_err4
	beq	$t0, $zero, print_err

#
# Third val valid, store in buildings array
#
	sw	$t9, 0($s7)
	addi	$s7, $s7, 4
	addi	$s1, $s1, -3
	j	buildings_loop

buildings_loop_done:
#
# Read input done
#
	move	$v0, $s0	# store the board size in v0
	move	$v1, $s5	# store amount of buildings in v1
	lw	$ra, -4+M_FRAMESIZE($sp)
	addi	$sp, $sp, M_FRAMESIZE
	jr	$ra
	
#
# Name:		check_building_rows_cols
#
# Description:	Error checks the row/column inputs of the building
#		data stream and terminates the program if there is.
# Arguments:	$a0: the value to check
# Destroys:	$t1, $t0
#

check_building_rows_cols:
	move	$t1, $a0	# t1 holds value to check
	slt	$t0, $t1, $s0
	la	$a0, str_err4
	beq	$t0, $zero, print_err
	
	slt	$t0, $t1, $zero
	bne	$t0, $zero, print_err
	
	jr	$ra

#
# Name:		print_unsolved
# 
# Description:	Prints the unsolved version of the puzzle
#		based on the results of read_input
# Arguments:	$a0: board size $a1: num of fixed buildings
#

print_unsolved:
P_FRAMESIZE = 36
	addi	$sp, $sp, -P_FRAMESIZE
	sw	$ra, -4+P_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	move	$s2, $a0	# $s2 holds board size n
	move	$s3, $a1	# $s3 holds num of fixed buildings
	la	$s0, hints	# $s0 holds base address of hints array
	la	$s1, buildings	# $s1 holds base address of buildings array
	
	move	$t0, $s2	# $t0 holds n for loop manipulation
	move	$t1, $s0	# $t1 holds hints array to loop over

	move	$t5, $zero	# $t5 holds row counter
	move	$t6, $zero	# $t6 holds column counter

	li	$v0, PRINT_STRING	# print the first gap
	la	$a0, str_space
	syscall
#
# Loop over hints and print north row on top
#
print_top_loop:
	beq	$t0, $zero, print_top_done
	lw	$t2, 0($t1)	# get the num in the hint array

	li	$v0, PRINT_STRING
	la	$a0, str_3space
	syscall

	beq	$t2, $zero, print_space_top
	

	li	$v0, PRINT_INT	# print the num in the array
	move	$a0, $t2
	syscall


space_printed_top:
	addi	$t0, $t0, -1
	addi	$t1, $t1, 4
	j	print_top_loop
	
#
# Finished printing top row of hints. Print rows...
#

print_top_done:
	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

	move	$a0, $s2
	jal	print_row

	move	$t8, $s2	
	addi	$t8, $t8, -1	# $t8 contains n - 1 for loop checking
	move	$s4, $s0	# $s4 contains hints array base
	
	mul	$t9, $s2, 4	# $t9 contains 4n
	add	$s4, $s4, $t9	# $s4 holds base adr of east side
	add	$s5, $s4, $t9	# $s5 holds base adr of south side
	add	$s6, $s5, $t9	# $s6 holds base adr of west side

#
# Get ready to loop through rows and print hints
#
print_mid_init:
	move	$t0, $s2	# $t0 contains n
	lw	$a0, 0($s6)	# pass element of west array 
	jal	print_hint_val	# print first val of west hint
	addi	$s6, $s6, 4

	move	$t6, $zero	# reset column counter to 0

	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall

#
# Print hints in rows, then vertical bars, then set values
#
print_mid_loop:
	beq	$t0, $zero, print_mid_loop_done
	li	$v0, PRINT_STRING
	la	$a0, str_vertical
	syscall
	la	$a0, str_space
	syscall

	move	$a0, $t5
	move	$a1, $t6
	move	$a2, $s3
	jal	check_and_print_building

	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall

	addi	$t6, $t6, 1	# increment column by 1
	addi	$t0, $t0, -1
	j	print_mid_loop
#
# Change values and go back to print_mid_loop for remaining rows
#
print_mid_loop_done:
	li	$v0, PRINT_STRING
	la	$a0, str_vertical
	syscall
	la	$a0, str_space
	syscall

	lw	$a0, 0($s4)	# print next value of east hints
	jal	print_hint_val
	addi	$s4, $s4, 4
	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

	beq	$t8, $zero, print_rows_done
	move	$a0, $s2
	jal	print_row

	addi	$t8, $t8, -1
	addi	$t5, $t5, 1	# increment row counter by 1
	j	print_mid_init

print_rows_done:
	move	$a0, $s2	# print the last row
	jal	print_row
	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall

	move	$t0, $s2	# $t0 holds n
#
# Board is printed with hints on all sides but south: print south hints
#
print_bottom_loop:
	beq	$t0, $zero, print_bottom_done

	li	$v0, PRINT_STRING
	la	$a0, str_3space
	syscall

	lw	$t1, 0($s5)
	beq	$t1, $zero, print_space_bottom 	# print bottom hint unless zero
	li	$v0, PRINT_INT
	move	$a0, $t1
	syscall

	

space_printed_bottom:
	addi	$t0, $t0, -1
	addi	$s5, $s5, 4
	j	print_bottom_loop

print_bottom_done:
	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

#
# print_unsolved done!
#
print_unsolved_done:
	lw	$ra, -4+P_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, P_FRAMESIZE
	jr	$ra

print_space_top:
	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall
	j	space_printed_top

print_space_bottom:
	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall
	j	space_printed_bottom

#
# Name:		check_and_print_building
#
# Description:	Checks if a building is at given row/column from
#		fixed buildings array.  If there is, print height.
#		If not, print " "
# Arguments:	$a0: row $a1: column $a2: num of fixed vals
#
check_and_print_building:
CP_FRAMESIZE = 36
	addi	$sp, $sp, -CP_FRAMESIZE
	sw	$ra, -4+CP_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s0, buildings	# $s0 holds base of buildings array
	move	$s1, $a0	# $s1 holds the row to check
	move	$s2, $a1	# $s2 holds the column to check
	move	$s3, $a2	# $s3 holds the num of fixed vals

	move	$s5, $s3	# $s5 holds num of fixed vals for looping

check_spots_loop:
	beq	$s5, $zero, checked_all_spots
	lw	$s4, 0($s0)		# $s4 holds row from building array
	beq	$s4, $s1, check_found_row	# if row checked = row in array found row

	addi	$s5, $s5, -1
	addi	$s0, $s0, 12		# move s0 to next row spot
	j	check_spots_loop

#
# Found row, now check for column
#
check_found_row:
	addi	$s0, $s0, 4		# point s0 to columns
	lw	$s6, 0($s0)

	beq	$s6, $s2, check_found_column
	addi	$s0, $s0, 8
	addi	$s5, $s5, -1
	j	check_spots_loop

check_found_column:
	addi	$s0, $s0, 4
	li	$v0, PRINT_INT
	lw	$a0, 0($s0)	# Found it! print building height
	syscall
	j	check_and_print_done

checked_all_spots:
	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall
	j	check_and_print_done

#
# Check and print done!
#
check_and_print_done:
	lw	$ra, -4+CP_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, CP_FRAMESIZE
	jr	$ra


#
# Name:		print_hint_val
#
# Description:	Prints a hint value to the board, blank if 0
# Arguments:	$a0: the value to print
#
print_hint_val:
	beq	$a0, $zero, print_hint_zero
	li	$v0, PRINT_INT
	syscall
	jr	$ra

print_hint_zero:
	li	$v0, PRINT_STRING
	la	$a0, str_space
	syscall
	jr	$ra

#
# Name:		print_row
#
# Description:	print a row for a board
# Arguments:	$a0: size of the board
# Destroys:	$t0
#
print_row:
	move	$t0, $a0		# $t0 holds board size
	li	$v0, PRINT_STRING
	la	$a0, str_2space
	syscall
	la	$a0, str_plus
	syscall
print_row_loop:
	beq	$t0, $zero, print_row_loop_done
	la	$a0, str_line
	syscall

	la	$a0, str_plus
	syscall

	addi	$t0, $t0, -1
	j	print_row_loop

print_row_loop_done:
	
	la	$a0, newline
	syscall
	jr	$ra

#
# Name:		print_banner
#
# Description:	Prints the initial banner
# Arguments:	n/a
#

print_banner:
	li	$v0, PRINT_STRING	# print banner
	la	$a0, str_init_top
	syscall

	la	$a0, str_init_mid
	syscall

	la	$a0, str_init_top
	syscall

	la	$a0, newline
	syscall
	jr	$ra


#
# Name:		print_err
# 
# Description:	Prints error 1 and terminates the program.
# Arguments:	$a0: address of error to print out
#

print_err:
	lw	$ra, -4+M_FRAMESIZE($sp)
	addi	$sp, $sp, M_FRAMESIZE

	li	$v0, PRINT_STRING
	syscall
	j	main_done

#
# Name:		print_arr
# 
# Description:	Prints an integer array.
# Arguments:	$a0: address of base of array
#		$a1: size of array
# Destroys:	$t0, $t1, $t2
#

print_arr:
	mul	$t0, $a1, 4	# $t0 holds length of arr x4
	move	$t1, $a0	# $t1 holds base address
	move	$t2, $zero	# $t2 holds counter

	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall

print_arr_loop:
	beq	$t2, $t0, print_arr_done
	li	$v0, PRINT_INT
	lw	$a0, 0($t1)
	syscall
	addi	$t1, $t1, 4
	addi	$t2, $t2, 4
	j	print_arr_loop

print_arr_done:
	li	$v0, PRINT_STRING
	la	$a0, newline
	syscall
	jr 	$ra


# File:		solve_skyscrapers.asm
# Author:	Colin Allen
#
# Description:	Solves a skyscraper puzzle
#
# Arguments:	$a0: board dimensions
#		$a1: number of fixed buildings
# Returns:	$v0: 0 if impossible 1 if solved
#

#
# Globals
#
	.globl 	solve_skyscrapers
	.globl 	hints
	.globl	buildings
	.globl	board_size
	.globl	num_fixed_buildings
	.globl	print_arr

#
# Constants for syscalls
#
PRINT_INT	= 1
PRINT_STRING	= 4

#
# Data areas
#
	.data
	.align 0

#
# Useful strings for debugging
#
str_newline:	.asciiz "\n"

	.align 	2
#
# Array to construct sorted rowcol
#
sorted_rowcol_arr: .word 0, 0, 0, 0, 0, 0, 0, 0


	.text

solve_skyscrapers:
A_FRAMESIZE = 40

#
# Save registers ra and s0 - s7 on the stack.
#
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)
	
	jal	solve
	beq	$v0, $zero, solve_return_0
	li	$v0, 1

	j	solve_skyscrapers_done

solve_return_0:
	li	$v0, 0
	j	solve_skyscrapers_done


#
# Name:		solve
#
# Description:	Solves the board.
# Returns:	$v0: 0 if no solution 1 if there is
#
solve:
	addi	$sp, $sp, -24
	sw	$ra, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	li	$s0, -1		# $t0 is row
	li	$s1, -1		# $t1 is col
	li	$s2, 1		# $t2 is isEmpty boolean

	move	$s3, $zero	# $t3 is i

	la	$s4, board_size
	lw	$s4, 0($s4)	# $t4 is n

solve_find_empty_loop1:
	beq	$s3, $s4, solve_find_empty_loop1_done
	move	$t5, $zero	# $t5 is j
solve_find_empty_loop2:
	beq	$t5, $s4, solve_find_empty_loop2_done
	
	move	$a0, $s3
	move	$a1, $t5
	jal	lookup_building
	beq	$v0, $zero, solve_no_building_lookup
	addi	$t5, $t5, 1
	j	solve_find_empty_loop2

solve_no_building_lookup:
	move	$s0, $s3
	move	$s1, $t5
	move	$s2, $zero
	j	solve_find_empty_loop2_done

solve_find_empty_loop2_done:
	beq	$s2, $zero, solve_find_empty_loop1_done
	addi	$s3, $s3, 1
	j	solve_find_empty_loop1
solve_find_empty_loop1_done:
	bne	$s2, $zero, solve_return_true
#
# If here, row/col are set to next empty space.
# Start backtracking...
#
	li	$s3, 1		# $t3 is num
	addi	$s4, $s4, 1	# $t4 is n + 1

solve_backtrack_loop:
	beq	$s3, $s4, solve_backtrack_loop_done
	move	$a0, $s0
	move	$a1, $s1
	move	$a2, $s3
	jal	is_safe
	bne	$v0, $zero, solve_backtrack_spot_safe
	addi	$s3, $s3, 1
	j	solve_backtrack_loop

solve_backtrack_spot_safe:
	move	$a0, $s0
	move	$a1, $s1
	move	$a2, $s3
	jal	add_building

	jal	solve
	bne	$v0, $zero, solve_return_true
	la	$t9, num_fixed_buildings
	lw	$t8, 0($t9)
	addi	$t8, $t8, -1
	sw	$t8, 0($t9)

	addi	$s3, $s3, 1
	j	solve_backtrack_loop

solve_backtrack_loop_done:
	li	$v0, 0
	j	solve_done

solve_return_true:
	li	$v0, 1
	j	solve_done

solve_done:
	lw	$ra, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, 24
	jr	$ra



#
# Name:		is_safe
#
# Description:	decides if a building is safe if it were placed 
#		at the given space.
# Arguments:	$a0: row $a1: col $a2: val
# Returns:	$v0: 0 if unsafe 1 if safe
# Destroys:	$t0 -> $t4
#
is_safe:
	addi	$sp, $sp, -16
	sw	$ra, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	move	$s0, $a0	# $s0 holds row
	move	$s1, $a1	# $s1 holds col
	move	$s2, $a2	# $s2 holds val

	jal	check_for_rowcol_repeat
	bne	$v0, $zero, not_safe

	move	$a0, $s0
	move	$a1, $s1
	move	$a2, $s2
	jal	check_row_full
	bne	$v0, $zero, not_safe

	move	$a0, $s0
	move	$a1, $s1
	move	$a2, $s2
	jal	check_col_full
	bne	$v0, $zero, not_safe

	li	$v0, 1
	j	is_safe_done
	

not_safe:
	li	$v0, 0

is_safe_done:
	lw	$ra, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, 16
	jr	$ra
	
#
# Name:		lookup_building
#
# Description:	lookup a building in the buildings arr
# Arguments:	$a0: row $a1: col
# Returns:	$v0: 1 if building there 0 if not.
#
lookup_building:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s0, buildings	# $s0 holds buildings
	move	$s1, $a0	# $s1 holds row
	move	$s2, $a1	# $s2 holds col
	la	$s3, num_fixed_buildings
	lw	$s3, 0($s3)	# $s3 holds num buildings
	# addi	$s3, $s3, -1	# $s3 holds num buildings - 1

lookup_building_loop:
	beq	$s3, $zero, lookup_building_false
	lw	$s4, 0($s0)	# $s4 holds row
	beq	$s4, $s1, lookup_building_row_found
	addi	$s0, $s0, 12
	addi	$s3, $s3, -1
	j	lookup_building_loop

lookup_building_row_found:
	addi	$s0, $s0, 4
	lw	$s4, 0($s0)	# $s4 holds found row's col
	beq	$s4, $s2, lookup_building_true

	addi	$s0, $s0, 4
	lw	$s4, 0($s0)	# $s4 holds found row's val
	beq	$s4, $zero, lookup_building_false

	addi	$s0, $s0, 4
	addi	$s3, $s3, -1
	j	lookup_building_loop
	
lookup_building_true:
	li	$v0, 1
	j	lookup_building_done

lookup_building_false:
	li	$v0, 0
	j	lookup_building_done

lookup_building_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra


#
# Name:		check_row_full
#
# Description:	Checks if a row is full and if it is,
#		checks if it adds up to the hint val.
# Arguments:	$a0 row to check $a1 col $a2 val
# Returns:	$v0: 0 if not full OR full and valid 
#		1 if full and invalid
#
check_row_full:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	move	$s4, $a0	# $s4 holds row to check

	jal	construct_sorted_row
	la	$t0, sorted_rowcol_arr

	mul	$t2, $a1, 4
	add	$t0, $t0, $t2
	sw	$a2, 0($t0)	# add val to spot

	la	$s0, board_size
	lw	$s0, 0($s0)	# $s0 contains n
	la	$s1, sorted_rowcol_arr	# $s1 contains sorted row arr


check_row_full_loop:
	beq	$s0, $zero, check_row_full_westeast
	lw	$s2, 0($s1)	# $s2 holds first element
	beq	$s2, $zero, check_row_full_return_0	#arr not full
	addi	$s0, $s0, -1
	addi	$s1, $s1, 4
	j	check_row_full_loop

#
# Row is full. Does it add up to hints right?
#
check_row_full_westeast:
	la	$s0, sorted_rowcol_arr	# $s0 contains row arr
	la	$s1, board_size
	lw	$s1, 0($s1)	# $s1 contains n

	move	$s6, $zero	# $s6 holds current max height seen
	move	$s7, $zero	# $s7 holds buildings seen

check_row_full_westeast_loop:
	beq	$s1, $zero, check_row_full_westeast_done
	lw	$s2, 0($s0)	# $s2 holds first val in row
	slt	$s3, $s6, $s2	# check if max height < this height
	bne	$s3, $zero, check_row_full_new_westeast_max
	addi	$s0, $s0, 4
	addi	$s1, $s1, -1
	j	check_row_full_westeast_loop

check_row_full_new_westeast_max:
	move	$s6, $s2
	addi	$s7, $s7, 1

	addi	$s0, $s0, 4
	addi	$s1, $s1, -1
	j	check_row_full_westeast_loop

#
# Have seen buildings, check against hint:
#
check_row_full_westeast_done:
	la	$t0, hints	# $t0 holds base of hints

	la	$s1, board_size
	lw	$s1, 0($s1)	# $s1 holds n
	mul	$s1, $s1, 12
	add	$t0, $t0, $s1	# $t0 points to base of west hints
	mul	$s1, $s4, 4	
	add	$t0, $t0, $s1	# $t0 points to hint in needed row
	lw	$s3, 0($t0)	# $s3 holds the hint to check against

	beq	$s3, $zero, check_row_full_eastwest

	beq	$s3, $s7, check_row_full_eastwest
	j	check_row_full_return_1	# row full and invalid

#
# west->east adds up how bout east->west
#
check_row_full_eastwest:
	la	$t0, sorted_rowcol_arr	# $t0 holds base of row arr
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n
	addi	$t1, $t1, -1	# $t1 holds n - 1
	mul	$t1, $t1, 4	# $t1 holds displacement amt
	add	$t0, $t0, $t1 	# $t0 points to val of last row
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n

	move	$s6, $zero	# reset max height seen
	move	$s7, $zero	# reset current buildings seen

check_row_full_eastwest_loop:
	beq	$t1, $zero, check_row_full_eastwest_done
	lw	$t2, 0($t0)	# $t2 holds first rows val
	slt	$t3, $s6, $t2	# check if max height < this height
	bne	$t3, $zero, check_row_full_new_eastwest_max
	addi	$t0, $t0, -4
	addi	$t1, $t1, -1
	j	check_row_full_eastwest_loop

check_row_full_new_eastwest_max:
	move	$s6, $t2
	addi	$s7, $s7, 1

	addi	$t0, $t0, -4
	addi	$t1, $t1, -1
	j	check_row_full_eastwest_loop

#
# If here, east->west vals added now check against hint
#
check_row_full_eastwest_done:
	la	$s0, hints	# $s0 holds base of north hint
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n
	mul	$t1, $t1, 4
	add	$t2, $s0, $t1	# $t2 holds base of east hints
	mul	$t1, $s4, 4	# $t1 holds displacement amt for hint
	add	$t2, $t2, $t1	# $t2 points to hint in needed row
	lw	$t3, 0($t2)	# $t3 holds hint to check against

	beq	$t3, $zero, check_row_full_return_0

	beq	$t3, $s7, check_row_full_return_0

check_row_full_return_1:
	li	$v0, 1
	j	check_row_full_done

check_row_full_return_0:
	li	$v0, 0
	j	check_row_full_done

check_row_full_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra


#
# Name:		check_col_ful
#
# Description:	Checks if a col is full and if it is,
#		checks if it adds up to hint val.
# Arguments:	$a0 row $a1 col $a2 val
# Returns:	$v0 0 if not full OR ful and valid
#		1 if full and invalid
#
check_col_full:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s4, buildings	# $s4 contains buildings arr
	la	$s0, hints	# $s0 holds hints arr

	move	$s1, $zero	# $s1 holds current MAX HEIGHT seen
	move	$s3, $zero	# $s3 holds buildings seen

	move	$s5, $a0	# $s5 holds row
	move	$s6, $a1	# $s6 holds col
	move	$s7, $a2	# $s7 holds val

	move	$a0, $s6
	jal	construct_sorted_col
	la	$t0, sorted_rowcol_arr	# $t0 holds sorted col

	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n
	
	mul	$t2, $s5, 4
	add	$t0, $t0, $t2
	sw	$s7, 0($t0)	# add val to spot in sorted ar
	mul	$t2, $t2, -1
	add	$t0, $t0, $t2	# reset sorted arr

check_col_full_northsouth_loop:
	beq	$t1, $zero, check_col_full_northsouth_done
	lw	$t2, 0($t0)	# $t2 holds first cols' val
	beq	$t2, $zero, check_col_full_return_0
	slt	$t3, $s1, $t2	# check if max height < this height
	bne	$t3, $zero, check_col_full_northsouth_new_max
	addi	$t0, $t0, 4
	addi	$t1, $t1, -1
	j	check_col_full_northsouth_loop

check_col_full_northsouth_new_max:
	move	$s1, $t2	# set new max height
	addi	$s3, $s3, 1	# add 1 to buildings seen

	addi	$t0, $t0, 4
	addi	$t1, $t1, -1
	j	check_col_full_northsouth_loop

#
# $s3 holds buildings seen from north->south. Check
# against hints...
#
check_col_full_northsouth_done:
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n
	move	$t2, $s0
	mul	$t1, $s6, 4
	add	$t2, $t2, $t1	# $t2 points to hint in needed col
	lw	$t3, 0($t2)	# $t3 holds the hint to check against

	beq	$t3, $zero, check_col_full_southnorth

	beq	$t3, $s3, check_col_full_southnorth
	j	check_col_full_return_1

check_col_full_southnorth:
	la	$t0, sorted_rowcol_arr	# $t0 holds base of col arr
	la	$t1, board_size
	lw	$t1, 0($t1)
	addi	$t1, $t1, -1	# $t1 holds n - 1
	mul	$t1, $t1, 4
	add	$t0, $t0, $t1	# $t0 points to val of last col
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n

	move	$s1, $zero	# reset max height
	move	$s3, $zero	# reset current buildings seen

#
# If here, north->south good. now south->north
#
check_col_full_southnorth_loop:
	beq	$t1, $zero, check_col_full_southnorth_done
	lw	$t2, 0($t0)	# $t2 holds last cols val
	slt	$t3, $s1, $t2	# check if max height < this height
	bne	$t3, $zero, check_col_full_southnorth_new_max
	addi	$t0, $t0, -4
	addi	$t1, $t1, -1
	j	check_col_full_southnorth_loop

check_col_full_southnorth_new_max:
	move	$s1, $t2
	addi	$s3, $s3, 1

	addi	$t0, $t0, -4
	addi	$t1, $t1, -1
	j	check_col_full_southnorth_loop

check_col_full_southnorth_done:
	la	$t1, board_size
	lw	$t1, 0($t1)	# $t1 holds n
	mul	$t1, $t1, 8
	add	$t2, $s0, $t1	# $t2 holds base of south hints
	mul	$t1, $s6, 4
	add	$t2, $t2, $t1	# $t2 points to hint in needed col
	lw	$t3, 0($t2)	# $t3 holds hint to check against

	beq	$t3, $zero, check_col_full_return_0

	beq	$t3, $s3, check_col_full_return_0
	j	check_col_full_return_1

check_col_full_return_1:
	li	$v0, 1
	j	check_col_full_done

check_col_full_return_0:
	li	$v0, 0
	j	check_col_full_done

check_col_full_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra



#
# Name:		add_building
#
# Description:	Adds a building to the buildings array.
# Arguments:	$a0: row $a1: column $a2: val
#
add_building:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s0, buildings	# $s0 contains buildings arr
	la	$s1, num_fixed_buildings
	lw	$s1, 0($s1)	# $s1 contains num_fixed_buildings
	mul	$s2, $s1, 12
	add	$s0, $s0, $s2	# $s0 points to end of buildings arr
	sw	$a0, 0($s0)
	sw	$a1, 4($s0)
	sw	$a2, 8($s0)	# store the row col and val
	addi	$s1, $s1, 1
	la	$s2, num_fixed_buildings
	sw	$s1, 0($s2)	# increment fixed buildings by 1

	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra



#
# Name:		construct_sorted_col
#
# Description:	Adds elements of a given col to 
#		sorted_rowcol_arr
# Arguments:	$a0: col
#
construct_sorted_col:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s0, sorted_rowcol_arr	# $s0 contains destination ar
	jal	clear_sorted_rowcols_arr
	la	$s1, num_fixed_buildings
	lw	$s1, 0($s1)		# $s1 contains num fixed buildings
	la	$s2, buildings		# $s2 contains buildings arr
	addi	$s2, $s2, 4		# move s2 to column
	move	$s7, $a0		# $s7 contains col to sort

construct_sorted_col_loop:
	beq	$s1, $zero, construct_sorted_col_done
	lw	$s3, 0($s2)	# $s3 contains col
	beq	$s3, $s7, construct_sorted_col_found
	addi	$s2, $s2, 12
	addi	$s1, $s1, -1
	j	construct_sorted_col_loop

construct_sorted_col_found:
	addi	$s2, $s2, -4	# move buildings arr to row
	lw	$s4, 0($s2)
	mul	$s4, $s4, 4	# $s4 contains index amnt for val
	addi	$s2, $s2, 8	# move buildings arr to val
	lw	$s5, 0($s2)	# $s5 contains val

	add	$s0, $s0, $s4	# move destination array to index
	sw	$s5, 0($s0)	# store the val
	mul	$s4, $s4, -1
	add	$s0, $s0, $s4	# move the array back

	addi	$s2, $s2, 8
	addi	$s1, $s1, -1
	j	construct_sorted_col_loop

construct_sorted_col_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra



#
# Name:		construct_sorted_row
#
# Description:	Adds elements of a given row to 
#		sorted_rowcol_arr
# Arguments:	$a0: row
#
construct_sorted_row:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)

	la	$s0, sorted_rowcol_arr	# $s0 contains destination ar
	jal	clear_sorted_rowcols_arr
	la	$s1, num_fixed_buildings
	lw	$s1, 0($s1)		# $s1 contains board size
	la	$s2, buildings		# $s2 contains buildings arr
	move	$s7, $a0		# $s7 contains row to sort

construct_sorted_row_loop:
	beq	$s1, $zero, construct_sorted_row_done
	lw	$s3, 0($s2)	# $s3 contains row
	beq	$s3, $s7, construct_sorted_row_found
	addi	$s2, $s2, 12
	addi	$s1, $s1, -1
	j	construct_sorted_row_loop

construct_sorted_row_found:
	addi	$s2, $s2, 4	# move buildings arr to column
	lw	$s4, 0($s2)
	mul	$s4, $s4, 4	# $s4 contains index amnt for val
	addi	$s2, $s2, 4	# move buildings arr to val
	lw	$s5, 0($s2)	# $s5 contains val

	add	$s0, $s0, $s4	# move destination array to index
	sw	$s5, 0($s0)	# store the val
	mul	$s4, $s4, -1
	add	$s0, $s0, $s4	# move the array back

	addi	$s2, $s2, 4
	addi	$s1, $s1, -1
	j	construct_sorted_row_loop

construct_sorted_row_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra

#
# Name: 	clear_sorted_rowcols_arr
# 
# Description:	Clears the sorted_rowcol_arr
#
clear_sorted_rowcols_arr:
	addi	$sp, $sp, -8
	sw	$ra, 4($sp)

	la	$t0, board_size
	lw	$t0, 0($t0)	# $t0 holds n
	la	$t1, sorted_rowcol_arr	# $t1 holds dest arr

clear_sorted_rowcols_loop:
	beq	$t0, $zero, clear_sorted_rowcols_done
	sw	$zero, 0($t1)
	addi	$t1, $t1, 4
	addi	$t0, $t0, -1
	j	clear_sorted_rowcols_loop

clear_sorted_rowcols_done:

	lw	$ra, 4($sp)
	addi	$sp, $sp, 8
	jr	$ra

#
# Name:		check_for_rowcol_repeat
#
# Description:	Checks if the given val already exists in
#		the given row/col
# Arguments:	$a0: row $a1: column $a2: val
# Return:	$v0: 0 or 1 boolean (1 if found 0 if not)
#
check_for_rowcol_repeat:
	addi	$sp, $sp, -A_FRAMESIZE
	sw	$ra, -4+A_FRAMESIZE($sp)
	sw	$s7, 28($sp)
	sw	$s6, 24($sp)
	sw	$s5, 20($sp)
	sw	$s4, 16($sp)
	sw	$s3, 12($sp)
	sw	$s2, 8($sp)
	sw	$s1, 4($sp)
	sw	$s0, 0($sp)
#
# First, check rows.
#
	move	$s0, $a0	# $s0 holds row to check
	move	$s1, $a1	# $s1 holds col to check
	move	$s2, $a2	# $s2 holds val to check

	move	$a0, $s0
	jal	construct_sorted_row
	la	$s3, sorted_rowcol_arr	# $s3 holds row arr
	
	la	$s4, board_size
	lw	$s4, 0($s4)	# $s4 holds n

check_for_row_repeat_loop:
	beq	$s4, $zero, check_for_row_repeat_done
	lw	$s5, 0($s3)	# $s5 holds val from rowcol arr
	beq	$s5, $s2, check_for_rowcol_repeat_found
	addi	$s3, $s3, 4
	addi	$s4, $s4, -1
	j	check_for_row_repeat_loop

#
# No repeats in rows.  Check columns
#
check_for_row_repeat_done:
	move	$a0, $s1
	jal	construct_sorted_col
	la	$s3, sorted_rowcol_arr	# $s3 holds col arr
	
	la	$s4, board_size
	lw	$s4, 0($s4)	# $s4 holds n

check_for_col_repeat_loop:
	beq	$s4, $zero, check_for_rowcol_no_repeat
	lw	$s5, 0($s3)	# $s5 holds val from col arr
	beq	$s5, $s2, check_for_rowcol_repeat_found
	addi	$s3, $s3, 4
	addi	$s4, $s4, -1
	j	check_for_col_repeat_loop

check_for_rowcol_no_repeat:
	li	$v0, 0
	j	check_for_rowcol_repeat_done

check_for_rowcol_repeat_found:
	li	$v0, 1
	j	check_for_rowcol_repeat_done

check_for_rowcol_repeat_done:
	
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE
	jr	$ra

#
# Restore registers ra and s0-s7 from the stack
# and return.
#
solve_skyscrapers_done:
	lw	$ra, -4+A_FRAMESIZE($sp)
	lw	$s7, 28($sp)
	lw	$s6, 24($sp)
	lw	$s5, 20($sp)
	lw	$s4, 16($sp)
	lw	$s3, 12($sp)
	lw	$s2, 8($sp)
	lw	$s1, 4($sp)
	lw	$s0, 0($sp)
	addi	$sp, $sp, A_FRAMESIZE

	jr	$ra		# Return to caller
	</code>
</pre>

