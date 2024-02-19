---
tags: security-tutorials
---

# How a computer stores and processes data

**In this short demo you will:** 
- Learn on how a computer stores and processes data


**Requirements:** 
- C Programming Language


For this exercise, assume that you have a physical or virtual Linux system with a C compiler installed, for example Linux GNU GCC.


Consider the following C program:

```C=
#include <stdio.h>
#include <string.h>

struct Student
{ int id;
  char province[3]; //ON, BC etc. + terminating null
  int age;
};
 int main( ){
  struct Student student1;
  // Assign values to structure variables
  student1.id = 100364168;
  strncpy(student1.province, "ON\0", sizeof(student1.province));
  student1.age = 18;
  printf("The size of struct member id is %ld bytes\n", sizeof(student1.id));
  printf("The size of struct member province is %ld bytes\n", sizeof(student1.province));
  printf("The size of struct member age is %ld bytes\n", sizeof(student1.age));
  printf("The size of struct Student is %ld bytes\n", sizeof(struct Student));
  return 0;
}
```

# Part I: Data Alignment

Answer the following questions based on the above C code, by filling in all of the blanks where indicated based on the output of the program (Table 1). Note that the size of a variable or data type is evaluated using sizeof operator in C Programming.

Table1. Storage sizes of variables or data types in the above C code

| Variable or data type              | Syze in bytes | 
|------------------------------------|---------------|
| student1.id (**100**)              |               |
| student1.province (**101**)            |               |
| student1.age (**102**)                 |               |
| Add lines **100**, **101**, and **102** (**103**)	 |               |
| Struct student (**104**)               |               |


**Question 1**  
Which is bigger? ____________

1. The size of struct Student (**104**) or
2. Subtotal of the sizes of all the members of struct Student (**103**).


# Part II: Get the Representation of Data Using GDB
In the following exercises we will look into how computers represent data based on the above code. Specially, you will use GDB commands to look into the memory to get the representations of all the members of struct Student. For example, if we declare an int variable:

**int a = 16;**

Then we can use the x command in GDB

(gdb) **x/4bt &a**  
**0xbffff56c: 00010000 00000000 00000000 00000000**

where the first item is memory address where an integer value (in 4 bytes) is stored, the second item is the content stored in the memory.

Now, answer the following questions after debugging the above C program, by filling in all of the blanks with the output of GDB commands (Table 2):

Table 2: Data presentation of variables in the above C code


| Variables         | Representation of values (in hexadecimal) (after a value is assigned to the variable) | 
|-------------------|---------------|
| student1.id       |   |
| student1.province |    |
| student1.age      |     |
| student1          |     |
