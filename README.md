# Table of content
- [Table of content](#table-of-content)
- [Introduction](#introduction)
	- [RoadMap](#roadmap)
- [Documentation](#documentation)
	- [Compilation](#compilation)
	- [Usage](#usage)
		- [General](#general)
		- [STDOUT test](#stdout-test)
		- [Malloc test](#malloc-test)
- [Examples](#examples)
	- [Basic](#basic)
	- [STDOUT](#stdout)
	- [malloc](#malloc)
	- [More examples](#more-examples)
---
# Introduction
This library **is not** the Libunit rush. But an improved version to write unit tests simply.
## RoadMap
- Norm checker
# Documentation
## Compilation
Compile the libunit with its Makefile and use the header (includes/libunit.h) in your tester code
## Usage
### General
- First, initialize your tester with the `init_tester` function by specifying its name
- Initialize a test list for each of the features you will test with the `init_tests_list`
- To add a test to a list, use add_test. Give it:
  - The list pointer
  - The name of the test
  - The line (no \n please) of code that will be print in the log file to help the user
  - Is a crash acceptable
  - The type of the test
  - The test function
  - The compare function or value (depending of the test type)
- When your test list is finished, add it to your test with `add_test_list`
- You can now start your tester with `launch_test`
---
### STDOUT test
You can test what is written on STDOUT. For this:
- In your test initialise a `t_stdout` with `init_stdout_test`.
- Do what you want that writes on the STDOUT
- Use `end_stdout_test` to finish reading
- You can now retrieve what has been written using the `output` fields of your `t_stdout`.
- Special thanks to y3ll0w42 and his `https://github.com/y3ll0w42/libunit` for his strategy to catch the stdout
---
### Malloc test
- To test malloc, first use `start_malloc_catcher` in your test (use it after any malloc only used for the test)
- To stop testing malloc, use `stop_malloc_catcher_and_print_leaks`. That will print leaks summary in log file
- To check non protected malloc, call `start_malloc_breaker` just after the `start_malloc_catcher` and stop it with `stop_malloc_breaker` just before the `stop_malloc_catcher_and_print_leaks`
- If the test crash with `malloc breaker` ans the same test doesn't crash without. That's you have a non protected malloc
- Warning, if you have `malloc_breaker` activated, no leaks gonna be detected. Indeed, the failed malloc not allocating memory, there will be no leaks
- You can use  `stop_malloc_catcher` to stop checking the malloc without checking the leaks
- To use malloc tests, you need to compile the executable with `-ldl -rdynamic`
- Special thanks to tmatis and his `https://github.com/tmatis/ft_mallocator` for the explanation of his strategy to catch the mallocs
# Examples
## Basic
This is what your main could look like:
```c
t_tester *tester = init_tester("Libft Tester");

t_tests_list *list1 = init_tests_list("ft_strlen");
add_test(list1, "ok", NULL, FALSE, INT_VALUE, &test_strlen, 2);
add_test(list1, "ko", "ft_strlen(\"abcdefghiklm\")", FALSE, INT_COMPARE, &test_strlen_ko, &compare_strlen_ko);
add_test_list(tester, list1);

t_tests_list *list2 = init_tests_list("ft_itoa");
add_test(list2, "ok", NULL, FALSE, STR_COMPARE, &test_itoa_ok, &compare_itoa_ok);
add_test(list2, "ko", NULL, FALSE, STR_VALUE, &test_itoa_ko, "12");
add_test_list(tester, list2);

launch_test(tester);
```
The result of such a test will look like this:
```
     Libft Tester     
----------------------

ft_strlen: ✔ ✗
ft_itoa:   ✔ ✗

Global result: (2/4)
```
Associated with a log file like this:
```
     Libft Tester     
----------------------

ft_strlen:
  > ok: OK
  > ko: KO
       EXCEPTED: [12]
       OBTAINED: [1]
	   Created by: [ft_strlen("abcdefghiklm")]

ft_itoa:
  > ok: OK
  > ko: KO
       EXCEPTED: [12]
       OBTAINED: [12\n]


Global result: (2/4)
```
## STDOUT
This main:
```c
#include "includes/libunit.h"

void ft_putendl(char *str)
{
	write(1, str, strlen(str));
}

char *test()
{
	t_stdout *ft_stdout = init_stdout_test();
	ft_putendl("Salut");
	end_stdout_test(ft_stdout);
	return ft_stdout->output;
}

int main()
{
	t_tester *tester = init_tester("STDOUT Tester");

	t_tests_list *list1 = init_tests_list("STDOUT");
	add_test(list1, "test ", "ft_putendl(\"Salut\")", FALSE, STR_VALUE, &test, "Salut\n");
	add_test_list(tester, list1);

	launch_test(tester);
}
```
Produce this output:
```
     STDOUT Tester     
-----------------------

STDOUT: ✗

Global result: (0/1)
```
And this log file:
```
     STDOUT Tester     
-----------------------

STDOUT:
  > test : KO
       EXCEPTED: [Salut\n]
       OBTAINED: [Salut]
       Created by: [ft_putendl("Salut")]


Global result: (0/1)
```
## Malloc
A test like that gonna crash becaus the malloc is not protected
```c
start_malloc_catcher();
start_malloc_breaker();
char *str = malloc(10);
str[0] = 'c';
stop_malloc_breaker();
stop_malloc_catcher_and_print_leaks();
```
A test like that gonna print a leaks summary in the log because the `str` is not free (`test_var` doesn't appear in log file because it's malloced before the `start_malloc_catcher`)
```c
char *test_var = malloc(10);
start_malloc_catcher();
char *str = malloc(10);
stop_malloc_catcher_and_print_leaks();
```
## More examples
You can find a main.c file that tests all the cases acepted by the lib. Yes, that's a tester for a tester, testerception (compile it with `gcc main.c libunit.a -ldl -rdynamic`)