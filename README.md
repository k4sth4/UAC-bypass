# UAC-bypass

If you're in Administrator group but are on Medium Mandatory Level, you can't run some commands and tool due to User Account Control. One should need to bypass UAC to get on High Mandatory Level, from there we can become SYSTEM. With UAC enabled we can't run tools like mimikatz, and sometime commands like changing administrator password etc.

# Exploitation

## Check for Permissions
```markdown
whoami /all
```
![OnPaste 20220608-165137](https://user-images.githubusercontent.com/106917304/172604971-be2d4e2f-11ee-43bd-b541-069500736a8d.png)
