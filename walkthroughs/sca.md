---
label: SCA Walkthrough
icon: code-square
order: 100
---

# Application Security Community Walkthrough

## Source Composition Analysis Walkthrough

Source composition analysis (SCA) tools scan third-party libraries used in a project, check them for known vulnerabilities, and generate output on findings. Using an intentionally vulnerable application called [OWASP NodeGoat](https://github.com/OWASP/NodeGoat) as a target, this tutorial will walkthrough using two free SCA tools:

- [npm audit](https://docs.npmjs.com/cli/v6/commands/npm-audit): npm is a software registry that developers can interact with using the command line interface (CLI). This CLI includes npm audit, which scans a project for vulnerable dependencies.
- [OSV-Scanner](https://osv.dev/): A tool that checks a project's dependencies for publicly disclosed vulnerabilities. If any are found, it generates a report that includes associated Common Vulnerability Enumeration (CVE) identifiers for each finding.

After running each tool, we'll analyze the results.

Lastly, we'll cover managing vulnerable dependencies and share an example of how to update some of them.

## Step 1 - Setup

!!!
If you are using a laptop at the SAINTCON AppSec booth, please skip this step and proceed to Step 2.
!!!

These instructions are for using the Linux distribution Ubuntu. If you're using macOS or Windows, please refer to each project's platform specific instructions:

- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [OSV-Scanner](https://google.github.io/osv-scanner/installation/)

For simplicity, this walkthrough will use the Downloads and Documents directories as destinations for the target code and SCA tools. For best practice recommendations on locations and environment variables, please refer to the above URLs.

### Step 1.1 - Setup npm

1. To check whether Node.js and npm are already installed, run these commands in the terminal:

```
node -v
npm -v
```

2. Install nodejs:

```
sudo apt install nodejs
```

3. Install npm:

```
sudo apt install npm
```

### Step 1.2 - Setup NodeGoat

You'll need git installed for this step. If it's not yet installed, then follow [these instructions first](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

1. Clone the repository locally to your computer. For example, you could create a new directory in your Documents directory:

```
mkdir /home/{your username here}/Documents/appsec
```

2. Switch to your destination directory and clone NodeGoat:

```
cd /home/{your username here}/Documents/appsec
git clone https://github.com/OWASP/NodeGoat.git
```

### Step 1.3 - Setup OSV-Scanner

1. Check whether the current version of Go is installed:

```
go version
```

2. Install Go:

```
sudo apt install golang-go
```

2. Install OSV Dependency-Check:

```
go install github.com/google/osv-scanner/cmd/osv-scanner@v1
```

## Step 2 - Run the SCA tools

### Step 2.1 - Run npm audit

1. Switch to the NodeGoat directory:

```
cd /home/{your username here}/Documents/appsec/NodeGoat
```

2. Run npm audit using the following command:

```
npm audit
```

3. Review the results of the scan in the output within the terminal, it should be similar to this:

```
117 vulnerabilities (3 low, 31 moderate, 56 high, 27 critical)
```

### Step 2.1 - Run OSV-Scanner

1. Run OSV-Scanner recursively against the local NodeGoat project directory:

```
osv-scanner -r /home/{your username here}/Documents/appsec/NodeGoat
```

3. Review the results. The default output will be a human readable table in your terminal with links to OSVs definition of the discovered vulnerabilites. You can explicitly choose a format for the output, OSV supports markdown, JSON, and SARIF. 

### Step 2.2 - Compare the results

1. Notice that npm audit and OSV-Scanner return different finding counts. From viewing the reports, what do you notice different between them?

## Step 3 - Fix vulnerabilities

npm audit includes an option that attempts to update identified vulnerable dependencies. Some findings can't be fixed this way due to factors like transitive dependencies.

Before running this in an actual development environment, consult with the developer(s) responsible for the project. Automatically updating dependencies should be accompanied with proper testing and validation to prevent errors due to breaking changes.

For more context on managing vulnerable dependencies, please refer to the [OWASP Vulnerable Dependency Management Cheat Sheet.](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html)

### Step 3.1 - Run npm audit fix

1. Switch back to the NodeGoat directory using the command from Step 2.1.1 and run [npm audit fix](https://docs.npmjs.com/cli/v10/commands/npm-audit):

```
npm audit fix
```

2. View the output details and notice that some don't have fixes and some are breaking changes. It should be similar to this:

```
100 vulnerabilities (3 low, 29 moderate, 46 high, 22 critical)
```


### Step 3.2 - Alternatively, Run OSV-Scanner Guided Remediation

OSV-Scanner has experimental remediation features that can be explored [here.] (https://google.github.io/osv-scanner/experimental/guided-remediation/)
