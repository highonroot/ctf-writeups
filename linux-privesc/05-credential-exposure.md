## What Credential Exposure Is

Credential exposure means storing credentials in locations that are readable by unintended users — plain text, config files, log files, txt files, shell history — these include user passwords, database credentials, API keys or tokens, SSH keys, web application credentials and more.

This is one of the easiest escalation paths — a user leaves their credentials carelessly.



## Why Credentials End Up Exposed

Human nature isn't malice — developers hardcode passwords for convenience — more often people are just careless.

Forgetting to clean up log or tmp files may store credentials unknowingly. Scripts using credentials in plain text are another common cause.



## Where Credentials Hide

- **`.bash_history`** — stores shell command history, including passwords passed as arguments
  
  **Example:**
  ```
  mysql -u root -pMySecretPassword
  ```

- **`.env` files** — store environment variables, often contain API keys and credentials
  
  **Example:**
  ```env
  DB_PASSWORD=supersecret123
  AWS_ACCESS_KEY_ID=AKIA...
  ```
  
- **Environment variables** — credentials passed at runtime, visible via `env`
  
  **Example:**
  ```
  export API_TOKEN=ghp_xxxxxxxxx
  ```
  
- **Config files** — database passwords, API keys or tokens
  
  **Example:**
  ```
  DB_PASSWORD = supersecret123
  ```
  
- **`/etc/fstab`** — mount configs, sometimes contain credentials
  
  **Example:**
  ```
  //fileserver/share /mnt/share cifs username=admin,password=secret123 0 0`
  ```

- **`/tmp`** — temporary files that may store credentials if not cleaned up
  
- **Scripts** — hardcoded credentials written directly for convenience
  
  **Example:**
  ```
  mysql -u root -pSuperSecretPass appdb
  ```
  
- **`/var/log`** — error logs that sometimes expose credentials in plain text
  
  **Example:**
  ```
  Login failed for user admin with password admin123
  ```
  
- **`~/.ssh/`** — private keys without passphrases allow direct access without a password
  
- **`crontab`** — scheduled scripts sometimes contain hardcoded credentials



## What Attackers Look For

- Passwords stored as plain text in scripts, shell history, or log files
- The string `"password"` or `"passwd"` in files
- API keys and tokens — specific patterns like `AKIA` for AWS keys
- Private key headers — `-----BEGIN RSA PRIVATE KEY-----`
- Connection strings — `mongodb://user:password@host`
- `.env` files anywhere on the filesystem
- World readable files containing sensitive data



## Commands

```bash
cat ~/.bash_history                   # list shell command history
env                                   # list environment variables
cat /etc/fstab                        # show mount config contents
find / -name "*.conf" 2>/dev/null     # list all .conf files
find / -name "*.env" 2>/dev/null      # list all .env files
find / -name "*.log" 2>/dev/null      # list all log files
grep -r "password" /etc 2>/dev/null   # filter lines containing "password"
grep -r "passwd" /home 2>/dev/null    # filter lines containing "passwd"
ls -la ~/.ssh/                        # list all files in ~/.ssh/ including hidden
cat ~/.ssh/id_rsa                     # display private key content
```



## What Comes Next

Credentials give you identity — but processes give you execution. Next, we look at how running processes and their execution context can be abused for escalation.


