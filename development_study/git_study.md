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
