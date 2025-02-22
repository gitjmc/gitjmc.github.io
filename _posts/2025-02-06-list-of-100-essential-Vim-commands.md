---
title: List of 100 essential Vim commands 
author:
  name: 
  link: 
date: 2025-02-06 18:35:00 -0400
categories: [CLI]
tags: [vim, edi]
render_with_liquid: false
---

Vim is a powerful text editor, but it has a steep learning curve. Below is a list of **100 essential Vim commands** categorized for easy reference. These commands will help you navigate, edit, and manage files efficiently in Vim.

---

### **Basic Navigation**
1. `h` - Move cursor left.
2. `j` - Move cursor down.
3. `k` - Move cursor up.
4. `l` - Move cursor right.
5. `w` - Move to the start of the next word.
6. `b` - Move to the start of the previous word.
7. `e` - Move to the end of the current word.
8. `0` - Move to the beginning of the line.
9. `$` - Move to the end of the line.
10. `gg` - Move to the first line of the file.
11. `G` - Move to the last line of the file.
12. `:n` - Move to line number `n` (e.g., `:10` goes to line 10).
13. `Ctrl + f` - Scroll forward one page.
14. `Ctrl + b` - Scroll backward one page.
15. `Ctrl + d` - Scroll down half a page.
16. `Ctrl + u` - Scroll up half a page.
17. `H` - Move to the top of the screen.
18. `M` - Move to the middle of the screen.
19. `L` - Move to the bottom of the screen.
20. `%` - Jump to the matching bracket `({[]})`.

---

### **Insert Mode**
21. `i` - Insert before the cursor.
22. `I` - Insert at the beginning of the line.
23. `a` - Insert after the cursor.
24. `A` - Insert at the end of the line.
25. `o` - Insert a new line below the current line.
26. `O` - Insert a new line above the current line.
27. `s` - Delete the character under the cursor and enter insert mode.
28. `S` - Delete the entire line and enter insert mode.
29. `Esc` - Exit insert mode and return to normal mode.

---

### **Editing Text**
30. `x` - Delete the character under the cursor.
31. `X` - Delete the character before the cursor.
32. `dd` - Delete the current line.
33. `dw` - Delete from the cursor to the start of the next word.
34. `d$` - Delete from the cursor to the end of the line.
35. `d0` - Delete from the cursor to the beginning of the line.
36. `yy` - Yank (copy) the current line.
37. `yw` - Yank from the cursor to the start of the next word.
38. `y$` - Yank from the cursor to the end of the line.
39. `p` - Paste after the cursor.
40. `P` - Paste before the cursor.
41. `u` - Undo the last action.
42. `Ctrl + r` - Redo the last undone action.
43. `.` - Repeat the last command.
44. `J` - Join the current line with the next line.
45. `r` - Replace the character under the cursor.
46. `R` - Enter replace mode (overwrite text).
47. `~` - Toggle case of the character under the cursor.
48. `>>` - Indent the current line.
49. `<<` - Unindent the current line.
50. `==` - Auto-indent the current line.

---

### **Search and Replace**
51. `/pattern` - Search forward for `pattern`.
52. `?pattern` - Search backward for `pattern`.
53. `n` - Repeat the last search forward.
54. `N` - Repeat the last search backward.
55. `:%s/old/new/g` - Replace all occurrences of `old` with `new` in the entire file.
56. `:%s/old/new/gc` - Replace with confirmation.
57. `:s/old/new/g` - Replace in the current line.
58. `:s/old/new/gc` - Replace in the current line with confirmation.
59. `*` - Search for the word under the cursor.
60. `#` - Search backward for the word under the cursor.

---

### **File Operations**
61. `:w` - Save the file.
62. `:wq` - Save and quit.
63. `:q` - Quit (if no changes were made).
64. `:q!` - Quit without saving changes.
65. `:w filename` - Save the file with a new name.
66. `:e filename` - Open another file in the current buffer.
67. `:r filename` - Insert the contents of another file into the current file.
68. `:saveas filename` - Save the file with a new name and switch to it.
69. `:bn` - Switch to the next buffer.
70. `:bp` - Switch to the previous buffer.
71. `:bd` - Close the current buffer.
72. `:ls` - List all open buffers.
73. `:e!` - Revert to the last saved version of the file.

---

### **Visual Mode**
74. `v` - Enter visual mode (character-wise).
75. `V` - Enter visual line mode.
76. `Ctrl + v` - Enter visual block mode.
77. `y` - Yank (copy) the selected text.
78. `d` - Delete the selected text.
79. `>` - Indent the selected text.
80. `<` - Unindent the selected text.
81. `~` - Toggle case of the selected text.

---

### **Advanced Editing**
82. `ciw` - Change the inner word (delete and enter insert mode).
83. `c$` - Change from the cursor to the end of the line.
84. `c0` - Change from the cursor to the beginning of the line.
85. `ct<char>` - Change until the specified character.
86. `cf<char>` - Change up to and including the specified character.
87. `diw` - Delete the inner word.
88. `dt<char>` - Delete until the specified character.
89. `df<char>` - Delete up to and including the specified character.
90. `gUw` - Uppercase the current word.
91. `guw` - Lowercase the current word.
92. `g~~` - Toggle case of the entire line.
93. `gqq` - Format the current line to a specific width.
94. `:set number` - Show line numbers.
95. `:set nonumber` - Hide line numbers.
96. `:set wrap` - Enable line wrapping.
97. `:set nowrap` - Disable line wrapping.
98. `:set paste` - Enable paste mode (prevents auto-indentation).
99. `:set nopaste` - Disable paste mode.
100. `:set spell` - Enable spell checking.

---

### **Tips for Using Vim Efficiently**
- Practice regularly to build muscle memory.
- Use `.vimrc` to customize Vim settings and create shortcuts.
- Learn to use macros (`q`) for repetitive tasks.
- Explore plugins like `NERDTree`, `vim-airline`, and `fzf` for enhanced functionality.

Let me know if you need further clarification or examples! 🚀

