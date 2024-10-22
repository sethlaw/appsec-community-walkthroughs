---
label: Semgrep Walkthrough
icon: code-square
order: 100
---

# Application Security Community Walkthrough
## Semgrep Walkthrough

This tutorial will walkthrough using the open-source version of [Semgrep Code](https://semgrep.dev/products/semgrep-code/), a `Fast, customizable, and developer-oriented SAST`. You will have the chance to run Semgrep against an intentionally-vulnerable, task managament application, [VTM](https://github.com/redpointsec/vtm) that is used in training developers about secure coding topics.

## Step 1 - Setup

Semgrep OSS is a command-line tool and requires a terminal to run. Open up a terminal window.

Confirm semgrep is installed on the local system by running `which semgrep`.

```
% which semgrep
/opt/homebrew/bin/semgrep
```

If you get message `semgrep not found`, flag down a volunteer so they can correct the problem. You can also fix the problem by running `pip install semgrep`

Now that we know semgrep is installed and available, open up a terminal window and make sure the `vtm` source code is included in the current user's home folder:

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
* `semgrep scan --config "p/django"`

```
┌──── ○○○ ────┐
│ Semgrep CLI │
└─────────────┘

                                                                                     
Scanning 59 files (only git-tracked) with 28 Code rules:
            
  CODE RULES
                                                                                     
  Language      Rules   Files          Origin      Rules                             
 ─────────────────────────────        ───────────────────                            
  python           27      17          Community      28                             
  <multilang>       1       9                                                        
                                       
...


┌──────────────┐
│ Scan Summary │
└──────────────┘
Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.
  Scan skipped: 13 files matching .semgrepignore patterns
  For a full list of skipped files, run semgrep with the --verbose flag.

Ran 28 rules on 26 files: 3 findings.

```

## Step 4 - Review Output

As with any automated tool, false positives are a possibility. Before adding SAST findings to the development backlog, we need to review and confirm the issues.

By default, Semgrep outputs findings to the terminal. These results can be saved to a file by specifying `semgrep -o out-file` at the end of the command.

In this case, we will review a SQL Injection finding that was identified with the first command.

* `cd saintcon-appsec-challenge-2023`
* `semgrep scan --config auto .`

 Scolling up you will see Semgrep provides a reference for which file the finding was located in. Here we see: `memeapp/controllers/users.py`. Semgrep gives us the following finding from the [_memeapp/controllers/users.py_](https://github.com/smanesse/saintcon-appsec-challenge-2023/blob/main/memeapp/controllers/users.py) source file: 

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

While this is a good start, we also want to confirm the issues through source code inspection. Opening up the _views.py_ file (located here: https://github.com/redpointsec/vtm/blob/main/taskManager/views.py , line# 805) we find that the call to `User.objects.raw` is found within the `forgot_password` function and does indeed take user input directly from the `email` parameter.

```python
@csrf_exempt
def forgot_password(request):
            
    if request.method == 'POST':
        t_email = request.POST.get('email')
    
        try:
            result = User.objects.raw("SELECT * FROM auth_user where email = '%s'" % t_email)

            if len(list(result)) > 0:
                reset_user = result[0]
                # Generate secure random 6 digit number
                res = ""
                nums = [x for x in os.urandom(6)]
                for x in nums:
                    res = res + str(x)
                   
                reset_token = res[:6]
                reset_user.userprofile.reset_token = reset_token
                reset_user.userprofile.reset_token_expiration = timezone.now() + datetime.timedelta(minutes=10)
                reset_user.userprofile.save()
                reset_user.save()
        
                #reset_user.email_user(
                        #"Reset your password",
                        #"You can reset your password at /taskManager/reset_password/. Use \"{}\" as your token. This link will only work for 10 minutes.".format(reset_token))

                messages.success(request, 'Check your email for a reset token')
                return redirect('/taskManager/reset_password')
            else:
                messages.warning(request, 'Check your email for a reset token')
        except User.DoesNotExist:
            messages.warning(request, 'Check your email for a reset token')

    return render(request, 'taskManager/forgot_password.html')
```

## Resources

- [Semgrep](https://semgrep.dev/) - Static Analysis Tool used to identify security issues in source code
- [Vulnerable Task Manager](https://github.com/redpointsec/vtm) - Intentionally vulnerable application we will target during this walkthrough.
