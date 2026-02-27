以下文档基于`cursor`

# macOS

已经存在文件夹`~/.cursor/skills-curosr`，这里有一些官方的技能文档，结构如下

```txt
skill_name
└── SKILL.md
```

文件夹名就是技能名，其中有一个`SKILL.md`

自己创建的技能文档不和官方文档放在一起，而是放在`~/.cursor/skills`下面 **这里需要校验**

让AI给出了一个替换字符串的例子

````markdown
---
name: text-replace
description: Replace text in files or strings using keyword substitution, regex patterns, template variable filling ({{var}}, ${var}), or batch replacements. Use when the user wants to find and replace text, fill in template placeholders, do bulk substitutions, or asks about replacing content in files.
---

# Text Replace

## Modes

**Keyword replacement**
```python
content = open('input.txt').read()
count = content.count('old')
result = content.replace('old', 'new')
open('output.txt', 'w').write(result)
print(f'Replaced {count} occurrences')
```

**Regex replacement**
```python
import re
content = open('input.txt').read()
result = re.sub(r'pattern', 'replacement', content, flags=re.IGNORECASE)
open('output.txt', 'w').write(result)
```

**Template variable filling** (supports `{{var}}` and `${var}`)

```python
template = open('template.txt').read()
variables = {'name': 'Alice', 'date': '2026-02-24'}
for key, value in variables.items():
    template = template.replace(f'{{{{{key}}}}}', value)
    template = template.replace(f'${{{key}}}', value)
open('output.txt', 'w').write(template)
```

**Batch replacement**

```python
replacements = [('old1', 'new1'), ('old2', 'new2')]
content = open('input.txt').read()
for old, new in replacements:
    content = content.replace(old, new)
open('output.txt', 'w').write(content)
```

## Guidelines
- Preview match count before replacing to avoid unintended changes
- For plain text input, return result directly in chat with replacement count
- For file input, write result to file and confirm path
- Ask for clarification on ambiguous matches (e.g. replacing `he` — should `she` be affected?)
````



