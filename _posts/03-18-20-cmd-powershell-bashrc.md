---
title: Create .bashrc files for Command Prompt and PowerShell in Windows
date: 2020-03-18 12:00:00 -500
categories: [development, tutorial]
tags: [windows,powershell,cmd]
---

# Create `.bashrc` files for Command Prompt and PowerShell in Windows

As someone who uses Linux a lot as my development workflow I've gotten used to the great power and flexibility of `.bashrc` files. If you're not familiar with Linux (really Unix-based systems in general) a `.bashrc` file allows you to customize certain things about your user shell by adding things like aliases, functions, different visuals, etc. A `.bashrc` file is really just a shell script that runs when you open a new shell so that everything is set up before you even hit the command line.

Since I do the vast majority of my development through WSL in Windows 10, I have a custom `.bashrc` file for each of my WSL distributions. However, I've often wanted the same functionality in Command Prompt (`cmd`) or PowerShell when I need to use those for Windows specific workflows. There are many different ways to set these types of things up, but after much experimentation across the web I've come up with two reliable ways to get the functionality I wanted.

## Setting up Command Prompt

For the `cmd` shell you need to create a `.cmd` file with whatever name you want (I called mine `bashrc.cmd`) and place it somewhere in your filesystem where it isn't going to move around. I like to keep things organized in my Documents folder so mine is in `%UserProfile%\Douments\Apps\cmd\bashrc.cmd`.

Below I have an example of a simple `bashrc.cmd` file which turns off printing the commands (line 1), sets the title of the `cmd` window (line 2), clears any text on the screen (line 3), and then prints some example text to the screen (line 4):

```bat
@echo off
title cmd
cls
echo "Hello World"
```

Now in order for this file to run when the command prompt is started we need to turn it on through the Registry (you should make sure to [backup your registry](https://support.microsoft.com/en-us/help/322756/how-to-back-up-and-restore-the-registry-in-windows) before making any changes, but the change we're making is not in any way dangerous):

- In order to open the Registry press `Win + R` and type in `regedit` and then press enter.
- Then navigate to `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Command Processor`.
- Right click on the `Command Processor` folder and click `New -&gt; String Value`.
- Enter `AutoRun` and press enter.
- Now double-click on `AutoRun` and type in the **full path** to your `bashrc.cmd` file and then click `OK`.

And that's it! No restart needed, just open up a new `cmd` prompt and (if you used the example file from above) you should see the title read `cmd` and the output say "Hello World". Feel free to adapt this file to your development needs! This file is really just a `.bat` file and so pretty much any command should work just fine.

## Setting up PowerShell

Setting up `.bashrc` functionality is even easier in PowerShell since it natively supports a very similar ability called [profiles](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7). There are a few different ways to set up a PowerShell profile depending on if you want the profile to be enabled for all users or not. Below, we will just set up a profile for a single user since I think that is much more common (unless you're an IT guy configuring development machines for a company).

In order to create a profile for your user, all you have to do is create a `profile.ps1` file in the directory `%UserProfile%\Documents\WindowsPowerShell\profile.ps1`. This will enable the profile for the current user and all PowerShell shells. An example of this type of file is below which makes the title of the shell windows "PowerShell" (line 1), clears any screen output (line 2), and then prints some example text to the screen (line 3):


```powershell
[System.Console]::Title = "PowerShell"
clear
echo "Hello World"
```

Now if you just try and open a PowerShell windows you'll most likely get an error about execution policies since by default you can't run PowerShell scripts that are unsigned. To get around this you need to open an elevated PowerShell prompt (run as Administrator) and run the below command:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

This command will allow local PowerShell scripts to execute even when unsigned (the profile script we created is unsigned) while refusing to run remote scripts if they are unsigned. The scope makes sure that this only is in effect for your current user. 

And now you're done! Go ahead and open up a new PowerShell prompt and (if you used the example file from above) you should see the title say "PowerShell" and the output say "Hello World". Feel free to modify the profile file however you'd like since it's just a PowerShell script!

## Conclusion

And that's it! These are really easy to set up and have a ton of functionality that you can play with. Be sure to let me know if anything above is incorrect or confusing so I can update the post and also let me know of any particular things you end up doing with these `.bashrc` files for Windows as I'd love to integrate new ideas into my own workflow!
