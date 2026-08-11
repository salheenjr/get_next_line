```markdown
*This project has been created as part of the 42 curriculum by <saalagor>.*

# get_next_line

## Description

**get_next_line** is a C programming project at 42. The objective is to write a function that reads and returns one line at a time from a file descriptor (`fd`), until reaching the End of File (EOF) or encountering an error.

Repeated calls to `get_next_line()` allow reading an entire text file or standard input (`stdin`) sequentially.

### Goal & Key Concepts
The project focuses on mastering low-level system calls and advanced C programming concepts:
* **File Descriptors & Low-Level I/O:** Using `read()` to stream data from files or `stdin`.
* **Static Variables:** Preserving state and data across multiple function calls.
* **Dynamic Memory Management:** Managing `malloc` allocations and preventing memory leaks using `free`.
* **Buffer Management:** Handling arbitrary buffer sizes (`BUFFER_SIZE`) specified at compile time.

---

### Algorithm Explanation & Justification

#### The Problem
The system call `read(fd, buffer, BUFFER_SIZE)` reads data in fixed-size blocks defined by `BUFFER_SIZE`. Because text lines are determined by newline characters (`\n`), a single call to `read()` will rarely align perfectly with the end of a line:
* It may read **less** than a full line (if `BUFFER_SIZE` is small).
* It may read **multiple lines** at once (if `BUFFER_SIZE` is large).

#### Selected Algorithm: Read-Extract-Update Workflow
To solve this, the algorithm uses a **static persistent buffer (stash)** to preserve unprocessed text across function calls. The process works in three distinct phases:

1. **Accumulate (`read_to_stash`):**
   * Read bytes from `fd` into a temporary buffer in chunks of `BUFFER_SIZE`.
   * Concatenate each chunk onto `stash` using `ft_strjoin`.
   * Continue reading until `stash` contains a newline character (`\n`), `read()` returns `0` (EOF), or an error occurs (`read()` returns `-1`).

2. **Extract (`extract_line`):**
   * Locate the first occurrence of `\n` in `stash`.
   * Allocate memory and copy characters from index `0` up to and including `\n` into a new string (`line`).
   * If EOF is reached with no trailing `\n`, extract all remaining characters.

3. **Clean Up (`update_stash`):**
   * Slice `stash` using `ft_substr` to remove the line that was just extracted.
   * Save the remaining characters (everything *after* `\n`) back into `stash` for the next `get_next_line` call.
   * Free the old `stash` memory. If no characters remain, set `stash = NULL`.

#### Multi-FD Algorithm Choice (Bonus)
To handle multiple file descriptors simultaneously without data corruption, `stash` is declared as an array of static pointers:
```c
static char *stash[MAX_FD];

```

Each open file descriptor (`fd`) accesses its own isolated static pointer (`stash[fd]`). This allows alternating calls between different files (e.g., `fd 3`, `fd 4`, `fd 5`) without losing the reading position or mixing buffers.

---

## Instructions

### Compilation

Compile the project using `cc` with `-Wall -Wextra -Werror` and specify the `-D BUFFER_SIZE=n` flag.

#### Mandatory Part:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c -o gnl

```

#### Bonus Part:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line_bonus.c get_next_line_utils_bonus.c main.c -o gnl_bonus

```

---

### Usage & Testing Example

Create a simple `main.c` file at the root of your repository to test reading a text file:

```c
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>
#include "get_next_line.h"

int main(void)
{
	int		fd;
	char	*line;
	int		line_number;

	fd = open("sample.txt", O_RDONLY);
	if (fd < 0)
	{
		printf("Error opening file\n");
		return (1);
	}

	line_number = 1;
	while ((line = get_next_line(fd)) != NULL)
	{
		printf("Line %d: %s", line_number++, line);
		free(line); // Memory allocated by get_next_line must be freed
	}

	close(fd);
	return (0);
}

```

#### Quick Execution Steps:

1. Create a sample text file:
```bash
echo -e "First line\nSecond line\nThird line" > sample.txt

```


2. Compile and run:
```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c -o gnl
./gnl

```


3. Check for memory leaks with Valgrind:
```bash
valgrind --leak-check=full --show-leak-kinds=all ./gnl

```



---

## Resources

### References & Documentation

* **`read(2)` Linux Manual Page:** `man 2 read`
* **File Descriptors:** [GNU C Library - File Descriptors](https://www.google.com/search?q=https://www.gnu.org/software/libc/manual/html_node/File-Descriptors.html)
* **Static Variables in C:** [GeeksforGeeks - Static Variables in C](https://www.geeksforgeeks.org/static-variables-in-c/)

### Use of AI Assistance

AI (Gemini) was used during this project for the following development tasks:

* **Conceptual Clarification:** Explaining Unix system calls, `ssize_t` return behaviors, and zero-indexing arithmetic for `buffer[bytes_read] = '\0'`.
* **Code Optimization & Norminette Refactoring:** Refactoring `ft_strjoin` and `ft_substr` to handle `NULL` pointers safely and reduce line count to adhere to 42 Norminette limits.
* **Edge-Case Analysis:** Identifying potential memory leaks during read errors (`bytes_read == -1`) and boundary checks for high file descriptor values (`MAX_FD`).
* **Test Suite Design:** Generating `main.c` test scripts for both mandatory and multi-FD bonus scenarios.

```

```
