# 1. first check git status

```bash
git status
```

upon checking git status it will throw error
![alt text](image-144.png)

and then do git init and then git status

```bash
git init
```

```bash
git status
```

![alt text](image-145.png)

now safe the repository first
![alt text](image-147.png)

- add gitignore
  ![alt text](image-146.png)

- now do

```bash
git add .
```

or

```bash
git add backend
```

and

```bash
git add client
```

![alt text](image-148.png)

![alt text](image-149.png)

# create a remote git repo

```bash
gh repo create mern-todo-dev --public --source=. --remote=origin
```

![alt text](image-150.png)

- git commit now

```bash
git commit -m "Change_Set_29032025_1916 : To do app first push to the git"
```

![alt text](image-151.png)
![alt text](image-152.png)

- now push to the branch main
  while pushing it will throw error as we have not removed the environment variable

```bash
git branch -m main
git push - u origin main
```

![alt text](image-153.png)

I am allowing it for testing purpose for now but we can also do put in git ignore file and then push it .

![alt text](image-154.png)
![alt text](image-155.png)

and now push is successful
![alt text](image-156.png)

# 3. now the file is in git hub

![alt text](image-157.png)
