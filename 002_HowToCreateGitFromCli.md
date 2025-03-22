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

- To add all the files for git use git add . or else use git add file_name

```bash
git add .
```

- setting the branch name - you can use master or main branch name for production

```bash
git branch -m "main"
```

or

```bash
git branch -m "master"
```

- commit the files . before this you can check git status as well what is about to be committed.

```bash
git commit -m "Provide the name of this push or write something comments or remarks about this push to git hub"
```

![alt text](image-18.png)

checking git status again ...

```bash
git status
```

![alt text](image-19.png)

- To push the git hub use the following command

```bash
git push -u origin main
```

![alt text](image-20.png)

- the git hub project has been created and pushed to git hub account

![alt text](image-21.png)

![alt text](image-22.png)

![alt text](image-23.png)

![alt text](image-24.png)

![alt text](image-25.png)

4. pushing the files again ..

- setting the branch to uat and we will merge in git hub directly

```bash
git branch -m uat
```

![alt text](image-26.png)

- checking the status

```bash
git status
```

![alt text](image-27.png)

- adding the files to the branch

```bash
git add .
```

- committing the files

```bash
git commit -m "This is second push for remaining files and the changes in the file"
```

- checkign the status again if anything missing or not done

```bash
git status
```

- pushing the committed files to the origin of the github

```bash
git push -u origin main
```
