
<b>Pattern 3: Use clear and descriptive variable names that accurately reflect their purpose, especially for boolean flags and parameters that affect code behavior.</b>

Example code before:
```
def extract_hunk_lines_from_patch(patch, file_name, line_start, line_end, side, remove_trailing_whitespace=True):
    # Function implementation
```

Example code after:
```
def extract_hunk_lines_from_patch(patch, file_name, line_start, line_end, side, remove_trailing_chars=True):
    # Function implementation
```

<details><summary>Relevant past discussions: </summary>

- https://github.com/Codium-ai/pr-agent-pro/pull/845#discussion_r1927083298
- https://github.com/Codium-ai/pr-agent-pro/pull/845#discussion_r1927089181
- https://github.com/Codium-ai/pr-agent-pro/pull/845#discussion_r1927085116
</details>
