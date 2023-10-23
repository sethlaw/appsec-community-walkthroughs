---
label: Running the Zap HUD tutorial
icon: code-square
order: 100
---

# Application Security Community Walkthrough

## Zap HUD
Zap's HUD functionality is an overlay that provides access to tools you can use while testing a site dynamically.

Some of the benefits of using the HUD over one of the static scripts are:
- First contact with a web application.  When you first encounter a web application you will not have existing scripts.  The HUD can help you do some basic analysis to determine what should be scripted specifically for the future.
- Authentication.  Since ZAP is a proxy and the HUD is an overlay for sites running through that proxy you can use the dynamic assessment capabilities to manually authenticate, especially when authentication requres 2FA or the authentication information is too sensitive to leave in a script.

### Zap HUD tutorial
Zap HUD already has a good built-in tutorial for what features it supplies and some practices exercises on using various features it provides. 

Rather than a feature by feature walkthrough this walkthrough will be how to access that tutorial.
Note that any flags in that tutorial are not part of the (Hacker's Challenge)[https://www.saintcon.org/con-hackerschallenge/]

#### Manual Explore
From the ZAP home screen, chose the option "manual explore".  This will take to to a screen such as this:
![](/static/zap/manual-explore.png)

##### Setup

###### URL to Explore
For URL to explore you will want to use the URL of a test site you have permission to access, never a third party or production site.

Anything running on your local host, such as the [SAINTCON 2023 AppSec challenge app](https://appsec.saintcon.community/challenge) or [Juice Shop](https://owasp.org/www-project-juice-shop/), should be fine.

###### Other Options
Choose "Enable HUD".

After choosing your favorite browser click "launch browser".
We recommend one that allows test automation, such as Chrome.

This will open up the Zap HUD:
![](/static/zap/HUD-start.png)

##### Tutorial
If you have never opened the HUD before you can select "Take the tutorial" rather than "Continue to your target".

If you have taken the tutorial before or are on a shared computer where someone else has taken it before you can reset the tutorial by:

Open the HUD settings
![](/static/zap/HUD-menu.png)

From here you can choose to reset the tutorial:
![](/static/zap/HUD-menu-retake-tutorial.png)
