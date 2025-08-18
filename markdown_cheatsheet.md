Here's a Markdown cheatsheet.

-----

## Headers

Headers are used for titles and section headings. The number of `#` symbols determines the heading level.

```markdown
# H1 Header
## H2 Header
### H3 Header
#### H4 Header
##### H5 Header
###### H6 Header
```

# H1 Header

## H2 Header

### H3 Header

#### H4 Header

##### H5 Header

###### H6 Header

-----

## Text Formatting

You can make text **bold**, *italic*, or both.

```markdown
**Bold text** or __Bold text__
*Italic text* or _Italic text_
***Bold and Italic text*** or ___Bold and Italic text___
~~Strikethrough text~~
```

**Bold text** or **Bold text**
*Italic text* or *Italic text*
***Bold and Italic text*** or ***Bold and Italic text***
\~\~Strikethrough text\~\~

-----

## Lists

Markdown supports ordered and unordered lists.

### Unordered List

Use `*`, `-`, or `+` followed by a space.

```markdown
* Item 1
* Item 2
  * Sub-item 2.1
- Item 3
+ Item 4
```

  * Item 1
  * Item 2
      * Sub-item 2.1

<!-- end list -->

  - Item 3

<!-- end list -->

  + Item 4

### Ordered List

Use a number followed by a period and a space.

```markdown
1. First item
2. Second item
3. Third item
```

1.  First item
2.  Second item
3.  Third item

-----

## Links and Images

To create a link, use `[Link text](URL)`. To insert an image, use `![Alt text](Image URL)`.

```markdown
[Google](https://www.google.com)
![A small gray cat](https://upload.wikimedia.org/wikipedia/commons/4/4d/Felis_silvestris_catus.jpg)
```

[Google](https://www.google.com)

-----

## Code

You can format code inline or in a block.

### Inline Code

Use backticks (`` ` ``) for inline code.

```markdown
This is `inline code`.
```

This is `inline code`.

### Code Blocks

Use three backticks (\`\`\`) to create a fenced code block. You can specify a language for syntax highlighting.

\<br\>

````markdown
```python
def hello_world():
  print("Hello, world!")
````

````
```python
def hello_world():
  print("Hello, world!")
````

-----

## Blockquotes

Use the `>` symbol to create a blockquote.

```markdown
> This is a blockquote.
> It can span multiple lines.
> > Nested blockquotes are also possible.
```

> This is a blockquote.
> It can span multiple lines.
>
> > Nested blockquotes are also possible.

-----

## Horizontal Rule

Use three or more `---`, `***`, or `___` on a line by themselves to create a horizontal rule.

```markdown
---
```

-----
