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
- Authentication.  Since ZAP is a proxy and the HUD is an overlay for sites running through that proxy you can use the dynamic assessment capabilities to manually authenticate, especially when authentication requires 2FA or the authentication information is too sensitive to leave in a script.

### Zap HUD tutorial
Zap HUD already has a good built-in tutorial for what features it supplies and some practices exercises on using various features it provides. 

Rather than a feature by feature walkthrough this walkthrough will be how to access that tutorial.

#### Reset

If you have taken the tutorial before or are on a shared computer where someone else has taken it before you will need to reset the tutorial by:

1. Open the "Options" menu
2. Select "HUD"
3. Click "Reset tutorial task"

![](/static/zap/Reset.png)

#### Starting the tutorial

##### Manual Explore
From the ZAP Quick Start screen, choose the option "Manual explore".  This will take to to a screen such as this:
![](/static/zap/manual-explore.png)



###### URL to Explore
For URL to explore you will want to use the URL of a test site you have permission to access, never a third party or production site.

Anything running on your local host, such as [Juice Shop](https://owasp.org/www-project-juice-shop/), should be fine.

###### Other Options
Choose "Enable HUD".

After choosing your favorite browser click "launch browser".
We recommend one that allows test automation, such as Chrome.

This will open up the Zap HUD:
![](/static/zap/HUD-start.png)

Select "Take the tutorial" rather than "Continue to your target".

This tutorial will walk you through the HUD and have you perform a couple of flag captures.  While the flags are not used in the Hacker's Challenge they are a good introduction into how to capture flags.


