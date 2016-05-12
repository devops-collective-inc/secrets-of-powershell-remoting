# Working with Endpoints (aka Session Configurations)

As you learned at the beginning of this guide, Remoting is designed to work with multiple different endpoints on a computer. In PowerShell terminology, each endpoint is a session configuration, or just a configuration. Each can be configured to offer specific services and capabilities, as well as having specific restrictions and limitations.

## Connecting to a Different Endpoint

When you use a command like Invoke-Command or Enter-PSSession, you normally connect to a remote computer's default endpoint. That's what we've done up to now. But you can see the other enabled endpoints by running Get-PSSessionConfiguration, as shown in figure 3.1.

![image042.png](images/image042.png)

Figure 3.1: Listing the installed endpoints

**Note:** As we pointed out in an earlier chapter, every computer will show different defaults endpoints. Our output was from a Windows Server 2008 R2 computer, which has fewer default endpoints than, say, a Windows 2012 computer.

Each endpoint has a name, such as "Microsoft.PowerShell" or "Microsoft.PowerShell32." To connect to a specific endpoint, add the -ConfigurationName parameter to your Remoting command, as shown in Figure 3.2.

![image043.png](images/image043.png)

Figure 3.2: Connecting to a specific configuration (endpoint) by name

## Creating a Custom Endpoint

There are a number of reasons to create a custom endpoint (or configuration):

- You can have scripts and modules auto-load whenever someone connects.
- You can specify a security descriptor (SDDL) that determines who is allowed to connect.
- You can specify an alternate account that will be used to run all commands within the endpoint - as opposed to using the credentials of the connected users.
- You can limit the commands that are available to connected users, thus restricting their capabilities.

There are two steps in setting up an endpoint: Creating a session configuration file which will define the endpoints capabilities, and then registering that file, which enables the endpoint and defines its configurations. Figure 3.3 shows the help for the New-PSSessionConfigurationFile command, which accomplishes the first of these two steps.

![image044.png](images/image044.png)

Figure 3.3: The New-PSSessionConfigurationFile command

Here's some of what the command allows you to specify (review the help file yourself for the other parameters):

- -Path: The only mandatory parameter, this is the path and filename for the configuration file you'll create. Name it whatever you like, and use a .PSSC filename extension.
- -AliasDefinitions: This is a hash table of aliases and their definitions. For example, @{Name='d';Definition='Get-ChildItem';Options='ReadOnly'} would define the alias d. Use a comma-separated list of these hash tables to define multiple aliases.
- -EnvironmentVariables: A single hash table of environment variables to load into the endpoint: @{'MyVar'='\\SERVER\Share';'MyOtherVar'='SomethingElse'}
- -ExecutionPolicy: Defaults to Restricted if you don't specify something else; use Unrestricted, AllSigned, or RemoteSigned. This sets the script execution policy for the endpoint.
- -FormatsToProcess and -TypesToProcess: Each of these is a comma-separated list of path and filenames to load. The first specifies .format.ps1xml files that contain view definitions, while the second specifies a .ps1xml file for PowerShell's Extensible Type System (ETS).
- -FunctionDefinitions: A comma-separated list of hash tables, each of which defines a function to appear within the endpoint. For example, @{Name='MoreDir';Options='ReadOnly';Value={ Dir | more }}
- -LanguageMode: The mode for PowerShell's script language. "FullLanguage" and "NoLanguage" are options; the latter permits only functions and cmdlets to run. There's also "RestrictedLanguage" which allows a very small subset of the scripting language to work - see the help for details.
- -ModulesToImport: A comma-separated list of module names to load into the endpoint. You can also use hash tables to specify specific module versions; read the command's full help for details.
- -PowerShellVersion: '2.0' or '3.0,' specifying the version of PowerShell you want the endpoint to use. 2.0 can only be specified if PowerShell v2 is independently installed on the computer hosting the endpoint (installing v3 "on top of" v2 allows v2 to continue to exist).
- -ScriptsToProcess: A comma-separated list of path and file names of scripts to run when a user connects to the endpoint. You can use this to customize the endpoint's runspace, define functions, load modules, or do anything else a script can do. However, in order to run, the script execution policy must permit the script.
- -SessionType: "Empty" loads nothing by default, leaving it up to you to load whatever you like via script or the parameters of this command. "Default" loads the normal PowerShell core extensions, plus whatever else you've specified via parameter. "RestrictedRemoteServer" adds a fixed list of seven commands, plus whatever you've specified; see the help for details on what's loaded.

**Caution:** Some commands are important - like Exit-PSSession, which enables someone to cleanly exit an interactive Remoting session. RestrictedRemoteServer loads these, but Empty does not.

- -VisibleAliases, -VisibleCmdlets, -VisibleFunctions, and -VisibleProviders: These comma-separated lists define which of the aliases, cmdlets, functions, and PSProviders you've loaded will actually be visible to the endpoint user. These enable you to load an entire module, but then only expose one or two commands, if desired.

**Note:** You can't use a custom endpoint alone to control which parameters a user will have access to. If you need that level of control, one option is to dive into .NET Framework programming, which does allow you to create a more fine-grained remote configuration. That's beyond the scope of this guide. You could also create a custom endpoint that only included proxy functions, another way of "wrapping" built-in commands and adding or removing parameters - but that's also beyond the scope of this guide.

Once you've created the configuration file, you're ready to register it. This is done with the Register-PSSessionConfiguration command, as shown in figure 3.4.

![image045.png](images/image045.png)

Figure 3.4: The Register-PSSessionConfiguration command

As you can see, there's a lot going on with this command. Some of the more interesting parameters include:

- -RunAsCredential: This lets you specify a credential that will be used to run all commands within the endpoint. Providing this credential enables users to connect and run commands that they normally wouldn't have permission to run; by limiting the available commands (via the session configuration file), you can restrict what users can do with this elevated privilege.
- -SecurityDescriptorSddl: This lets you specify who can connect to the endpoint. The specifier language is complex; consider using -ShowSecurityDescriptorUI instead, which shows a graphical dialog box to set the endpoint permissions.
- -StartupScript: This specifies a script to run each time the endpoint starts.

You can explore the other options on your own in the help file. Let's take a look at actually creating and using one of these custom endpoints. As shown in figure 3.5, we've created a new AD user account for SallyS of the Sales department. Sally, for some reason, needs to be able to list the users in our AD domain - but that's all she must be able to do. As-is, her account doesn't actually have permission to do so.

![image046.png](images/image046.png)

Figure 3.5: Creating a new AD user account to test

Figure 3.6 shows the creation of the new session configuration file, and the registration of the session. Notice that the session will auto-import the ActiveDirectory module, but only make the Get-ADUser cmdlet visible to Sally. We've specified a restricted remote session type, which will provide a few other key commands to Sally. We also disabled PowerShell's scripting language. When registering the configuration, we specified a "Run As" credential (we were prompted for the password), which is the account all commands will actually execute as.

![image047.png](images/image047.png)

Figure 3.6: Creating and registering the new endpoint

Because we used the -ShowSecurityDescriptorUI, we got a dialog box like the one shown in figure 3.7. This is an easier way of setting the permissions for who can use this new endpoint. Keep in mind that the endpoint will be running commands under a Domain Admin account, so we want to be very careful who we actually let in! Sally needs, at minimum, Execute and Read permission, which we've given her.

![image048.png](images/image048.png)

Figure 3.7: Setting the permissions on the endpoint

We then set a password for Sally and enabled her user account. Everything up to this point has been done on the DC01.AD2008R2.loc computer; figure 3.8 moves to that domain's Windows 7 client computer, where we logged in using Sally's account. As you can see, she was unable to enter the default session on the domain controller. But when she attempted to enter the special new session we set up just for her, she was successful. She was able to run Get-ADUser as well.

![image049.png](images/image049.png)

Figure 3.8: Testing the new endpoint by logging in as Sally

Figure 3.9 confirms that Sally has a very limited number of commands to play with. Some of these commands - like Get-Help and Exit-PSSession - are pretty crucial for using the endpoint. Others, like Select-Object, give Sally a minimal amount of non-destructive convenience for getting her command output to look like she needs. This command list (aside from Get-ADUser) is automatically set when you specify the "restricted remote" session type in the session configuration file.

![image050.png](images/image050.png)

Figure 3.9: Only eight commands, including the Get-ADUser one we added, are available within the endpoint.

In reality, it's unlikely that a Sales user like Sally would be running commands in the PowerShell console. More likely, she'd use some GUI-based application that ran the commands "behind the scenes." Either way, we've ensured that she has exactly the functionality she needs to do her job, and nothing more.

## Security Precautions with Custom Endpoints

When you create a custom session configuration file, as you've seen, you can set its language mode. The language mode determines what elements of the PowerShell scripting language are available in the endpoint - and the language mode can be a bit of a loophole. With the "Full" language mode, you get the entire scripting language, including script blocks. A script block is any executable hunk of PowerShell code contained within {curly brackets}. They're the loophole. Anytime you allow the use of script blocks, they can run any legal command - even if your endpoint used -VisibleCmdlets or -VisibleFunctions or another parameter to limit the commands in the endpoint.

In other words, if you register an endpoint that uses -VisibleCmdlets to only expose Get-ChildItem, but you create the endpoint's session configuration file to have the full language mode, then any script blocks inside the endpoint can use any command. Someone could run:

````
PS C:\> & { Import-Module ActiveDirectory; Get-ADUser -filter \* | Remove-ADObject }
````

Eek! This can be especially dangerous if you configured the endpoint to use a RunAs credential to run commands under elevated privileges. It's also somewhat easy to let this happen by mistake, because you set the language mode when you create the new session configuration file (New-PSSessionConfigurationFile), not when you register the session (Register-PSSessionConfiguration). So if you're using a session configuration file created by someone else, pop it open and confirm its language mode before you use it!

You can avoid this problem by setting the language mode to NoLanguage, which shuts off script blocks and the rest of the scripting language. Or, go for RestrictedLanguage, which blocks script blocks while still allowing some basic operators if you want users of the endpoint to be able to do basic filtering and comparisons.

Understand that this isn't a bug - the behavior we're describing here is by design. It can just be a problem if you don't know about it and understand what it's doing.

Note: Much thanks to fellow MVP Aleksandar Nikolic for helping me understand the logic of this loophole!



