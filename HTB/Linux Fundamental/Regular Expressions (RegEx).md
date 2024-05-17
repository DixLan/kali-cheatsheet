# Regular Expressions (RegEx)

1. `(a)` The round brackets are used to group parts of a regex. Within the brackets, you can define further patterns which should be processed together.
2. `[a-z]` The square brackets are used to define character classes. Inside the brackets, you can specify a list of characters to search for.
3. `{1,10}` The curly brackets are used to define quantifiers. Inside the brackets, you can specify a number or a range that indicates how often a previous pattern should be repeated.
4. `|` Also called the OR operator and shows results when one of the two expressions matches.
5. `.*` Also called the AND operator and displays results only if both expressions match.

## Examples :

```bash
grep -E "(my|false)" /etc/passwd
grep -E "my" /etc/passwd | grep -E "false"
```

## Exercises :

Here are some optional tasks to practice regex that can help us handle it better and more efficiently. For all exercises, we will use the `/etc/ssh/sshd_config` file.

1. Show all lines that do not contain the `#` character.
    ```bash
    grep -E "[a-z]" /etc/ssh/sshd_config | grep -E -v "(#)"
    ```
    (Select every line that contains a to z and show which doesn't contain `#`)

2. Search for all lines that contain a word that starts with `Permit`.
    ```bash
    grep -E "(Permit)" /etc/ssh/sshd_config
    ```

3. Search for all lines that contain a word ending with `Authentication`.
    ```bash
    grep -E "(*Authentication)" /etc/ssh/sshd_config
    ```

4. Search for all lines containing the word `Key`.
    ```bash
    grep -E "(Key)" /etc/ssh/sshd_config
    ```

5. Search for all lines beginning with `Password` and containing `yes`.
    ```bash
    grep -E "(Password.*yes)" /etc/ssh/sshd_config
    ```

6. Search for all lines that end with `yes`.
    ```bash
    grep -E -i "(* yes)" /etc/ssh/sshd_config | grep -E -v "(# )"
    ```
    (In the comment, we see this pattern `# blabla` so we exclude the `# ` and only keep settings)