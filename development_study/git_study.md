## git study

1.Git 保存用户名和密码

	git config credential.helper store
	
	参考
	$ man git | grep -C 5 password
	$ man git-credential-store
	
	这样保存的密码是明文的，保存在用户目录~的.git-credentials文件中
	$ file ~/.git-credentials
	$ cat ~/.git-credentials


EXAMPLES

The point of this helper is to reduce the number of times you must type your username or password. For example:

	$ git config credential.helper store
	$ git push http://example.com/repo.git
	Username: <type your username>
	Password: <type your password>

	[several days later]
	$ git push http://example.com/repo.git
	[your credentials are used automatically]


### git branch
```
Deleting local branches in Git
git branch -d feature/login
git branch -D branch branch2 --force

Deleting remote branches in Git
git branch -dr origin/feature/login

To delete a remote branch you need to push the delete:
git push remote --delete branch

The –delete option is newish, so if your git is old you can use the original syntax:
git push remote :branch


To delete a local branch
git branch -d the_local_branch

To remove a remote branch (if you know what you are doing!)
git push origin :the_remote_branch


If you get the error error: unable to push to unqualified destination: the_remote_branch The destination refspec neither matches an existing ref on the remote nor begins with refs/, and we are unable to guess a prefix based on the source ref. error: failed to push some refs to 'git@repository_name'

perhaps someone else has already deleted the branch. Try to synchronize your branch list with
git fetch -p 


```