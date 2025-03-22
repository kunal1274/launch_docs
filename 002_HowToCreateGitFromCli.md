# How to create git hub project from cli

![alt text](image-7.png)

![alt text](image-8.png)

![alt text](image-9.png)

Use Passkey or password as per your convenience -

![alt text](image-10.png)

I am using password method -

![alt text](image-11.png)

![alt text](image-12.png)

![alt text](image-13.png)

2. Initialize the repository

```bash
git init
```

![alt text](image-14.png)

- if git is not initialized then do the following

```bash
git status
```

![alt text](image-15.png)

```bash
git config --global --add safe.directory D:/NEXTARRATXEN/Launch-App-Docs
```

now git init is working .
![alt text](image-16.png)

3. Now create the repository from here directly through cli.

```bash
gh repo create launch_docs --public --source=. --remote=origin
```

![alt text](image-17.png)
