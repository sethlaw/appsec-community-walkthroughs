---
label: Semgrep Walkthrough
icon: code-square
order: 100
---

# Application Security Community Walkthrough
## Semgrep Walkthrough

This tutorial will walkthrough using the open-source version of [Semgrep Code](https://semgrep.dev/products/semgrep-code/), a `Fast, customizable, and developer-oriented SAST`. You will have the chance to run Semgrep against an intentionally-vulnerable, Meme generating application, [MemeApp](https://github.com/smanesse/saintcon-appsec-challenge-2023/tree/main) that is used in training developers about secure coding topics.

## Step 1 - Setup

Semgrep OSS is a command-line tool and requires a terminal to run. Open up a terminal window.

Confirm semgrep is installed on the local system by running `which semgrep`.

```
% which semgrep
/opt/homebrew/bin/semgrep
```

If you get message `semgrep not found`, flag down a volunteer so they can correct the problem. You can also fix the problem by running `pip install semgrep`

Now that we know semgrep is installed and available, open up a terminal window and make sure the `MemeApp` source code is included in the current user's home folder:

```
cd saintcon-appsec-challenge-2023
```

If this folder does not exist, clone vtm from GitHub using the command `git clone https://github.com/smanesse/saintcon-appsec-challenge-2023`.

## Step 2 - Run a SAST scan

Now that we have the code and tool available, run a scan using the default rules from the Semgrep team as follows: 

* `cd saintcon-appsec-challenge-2023`
* `semgrep scan --config auto .`

This will run analysis on the source code files that exist in the targeted directory and output something similar to the following:

```
┌──── ○○○ ────┐
│ Semgrep CLI │               
└─────────────┘               
                              
Scanning 59 files (only git-tracked) with:

                                      
...
                    
┌──────────────┐
│ Scan Summary │
└──────────────┘
Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.
  Partially scanned: 5 files only partially analyzed due to parsing or internal Semgrep errors
  Scan skipped: 13 files matching .semgrepignore patterns
  For a full list of skipped files, run semgrep with the --verbose flag.

Ran 471 rules on 46 files: 10 findings.


```

## Step 3 - Run a targeted SAST scan

One thing to note from the results is that the scan is attempting to analyze all available source files that it can match, including Python, JavaScript, XML, JSON, or anything else in that directory.

Given that VTM is a Django/Python application, we can scan specifically for known Django issues from the Semgrep registry (https://semgrep.dev/r). This is accomplished by running the following command:

* `cd saintcon-appsec-challenge-2023`
* `semgrep scan --config "p/flask"`

```
┌──── ○○○ ────┐
│ Semgrep CLI │
└─────────────┘

                                                                                     
Scanning 59 files (only git-tracked) with 17 Code rules:
            
  CODE RULES
  Scanning 17 files with 17 python rules.

                                       
...


┌──────────────┐
│ Scan Summary │
└──────────────┘
Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.
  Scan skipped: 12 files matching .semgrepignore patterns
  For a full list of skipped files, run semgrep with the --verbose flag.

Ran 17 rules on 17 files: 1 finding.


```

## Step 4 - Review Output

As with any automated tool, false positives are a possibility. Before adding SAST findings to the development backlog, we need to review and confirm the issues.

By default, Semgrep outputs findings to the terminal. These results can be saved to a file by specifying `semgrep -o out-file` at the end of the command.

In this case, we will review a SQL Injection finding that was identified with the first command.

* `cd saintcon-appsec-challenge-2023`
* `semgrep scan --config auto .`

 Scolling up you will see Semgrep provides a reference for which file the finding was located in. Semgrep gives us the following finding from the [_memeapp/controllers/users.py_](https://github.com/smanesse/saintcon-appsec-challenge-2023/blob/main/memeapp/controllers/users.py) source file: 

```
python.django.security.injection.tainted-sql-
       string.tainted-sql-string                    
          Detected user input used to manually   
          construct a SQL string. This is usually
          bad practice because manual            
          construction could accidentally result 
          in a SQL injection. An attacker could  
          use a SQL injection to steal or modify 
          contents of the database. Instead, use 
          a parameterized query which is         
          available by default in most database  
          engines. Alternatively, consider using 
          the Django object-relational mappers   
          (ORM) instead of raw SQL queries.      
          Details: https://sg.run/PbZp           
                                                 
           15┆ user = db_query(f"SELECT 
               rowid, password_hash FROM
               users WHERE              
               username='{username}'",  
               one=True)
```

While this is a good start, we also want to confirm the issues through source code inspection. Opening up the   [_memeapp/controllers/users.py_](https://github.com/smanesse/saintcon-appsec-challenge-2023/blob/main/memeapp/controllers/users.py) source file: , line# 15 we find that the  'user' query takes input from the user captured in the 'username' field and inserts it directly into the 'user' query. A classic example of a SQL injection vulnerability.

```python
@bp.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "GET":
        return redirect("/")
    username = request.form.get("username", None)
    password = request.form.get("password", None)
    user = db_query(f"SELECT rowid, password_hash FROM users WHERE username='{username}'", one=True)
    if user:
        user_id = user[0]
        password_hash = user[1]
        if cryptoutils.check_password(password, password_hash):
            response = redirect("/")
            set_session_token(user_id, response)
            return response
        else:
            return redirect("/login?error=Invalid+login")
    else:
        return redirect("/login?error=Invalid+login")

```


## Resources

- [Semgrep](https://semgrep.dev/) - Static Analysis Tool used to identify security issues in source code
- [MemeApp](https://github.com/smanesse/saintcon-appsec-challenge-2023/tree/main) - Intentionally vulnerable application we will target during this walkthrough.
