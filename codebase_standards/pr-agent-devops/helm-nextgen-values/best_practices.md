<b>Pattern 5: Use StringEnum for constant string values that are used in multiple places to ensure consistency and make future changes easier to manage.</b>

Example code before:
```
body_no_checkbox = comment_after.replace(line, '`[Generating...]`')
```

Example code after:
```
body_no_checkbox = comment_after.replace(line, StringEnum.GENERATING_SUGGESTIONS.value)
```

<details><summary>Relevant past discussions: </summary>

- https://github.com/Codium-ai/pr-agent-pro/pull/918#discussion_r1944208162
- https://github.com/Codium-ai/pr-agent-pro/pull/918#discussion_r1944256344
- https://github.com/Codium-ai/pr-agent-pro/pull/918#discussion_r1944259576
</details>
