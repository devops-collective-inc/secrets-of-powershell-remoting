# PowerShell, Remoting, and Security

Although PowerShell Remoting has been around since roughly 2010, many administrators and organizations are unable to take advantage of it, due in large part to outdated or uninformed security and risk avoidance policies. This chapter is designed to help address some of those by providing some honest technical detail about how these technologies work. In fact, they present significantly less risk than many of the management and communications protocols already in widespread use - those older protocols benefit primarily from being "grandfathered" into policies and never closely examined.

## Neither PowerShell nor Remoting are a "Back Door" for Malware

This is a major misconception. Keep in mind that, by default, PowerShell does not execute scripts. When it does so, it can only execute commands that the executing user has permission to run - it does not execute anything under a super-privileged account, and it bypasses neither existing permissions nor security. In fact, because PowerShell is based upon .NET, it's unlikely any malware author would even bother to utilize PowerShell. Such an attacker could simply call on .NET Framework functionality directly, and much more easily.

By default, PowerShell Remoting enables only Administrators to even connect, and once connected they can only run commands they have permission to run - with no ability to bypass permissions or underlying security. Unlike past tools which ran under a highly-privileged account (such as LocalSystem), PowerShell Remoting executes commands by impersonating the user who submitted the commands.

Bottom line: Because of the way it works, PowerShell Remoting does not allow any user, authorized or not, to do anything that they could not do through a dozen other means - including logging onto the console. Whatever protections you have in place to prevent those kinds of attacks (such as appropriate authorization and authentication mechanisms) will also protect PowerShell and Remoting. If you allow Administrators to log on to server consoles - either physically or via Remote Desktop - you have far greater security exposure than you do through PowerShell Remoting.

Further, PowerShell offers a better opportunity to restrict even Administrators. A Remoting endpoint (or session configuration) can be modified to allow only specified users to connect to it. Once connected, the endpoint can further restrict the commands that those users can execute. This provides a much better opportunity for delegated administration. Rather than having Administrators log onto consoles and do whatever they please, you can have them connect to restricted, secured endpoints and only complete those specific tasks that the endpoint permits.

## PowerShell Remoting is Not Optional

As of Windows Server 2012, PowerShell Remoting is enabled by default and is mandatory for server management. Even when running a graphical management console locally on a server, the console still "goes out" and "back in" via Remoting to accomplish its tasks. Without Remoting, server administration is impossible. Organizations are therefore well-advised to start immediately finding a way to include Remoting in their permitted protocols. Otherwise, critical services will not be able to be managed, even through Remote Desktop or directly on the server console.

This approach actually helps better secure the data center. Because local administration is exactly the same as remote administration (via Remoting), there's no longer any reason to physically or remotely access server consoles. The consoles can thus remain more locked down and secured, and Administrators can stay out of the data center entirely.

## Remoting Does Not Transmit or Store Credentials

By default, Remoting uses Kerberos, an authentication protocol that does not transmit passwords across the network. Instead, Kerberos relies on passwords as an encryption key, ensuring that passwords remain safe. Remoting can be configured to use less-secure authentication protocols (such as Basic), but can also be configured to require certificate-based encryption for the connection.

Further, Remoting never stores credentials in any persistent storage by default. A Remote machine never has access to a user's credentials; it has access only to a delegated security token (a Kerberos "ticket"). That is stored in volatile memory which cannot, by OS design, be written to disk - even to the OS page file. The server presents that token to the OS when executing commands, causing the command to be executed with the original invoking user's authority - and nothing more.

## Remoting Uses Encryption

Most Remoting-enabled applications apply their own encryption to their application-level traffic sent over Remoting. However, Remoting can also be configured to use HTTPS (certificate-encrypted connections), and can be configured to make HTTPS mandatory. This encrypts the entire channel using high-level encryption, while also ensuring mutual authentication of both client and server.

## Remoting is Security-Transparent

As stated, Remoting neither adds anything to, nor takes anything away from, your existing security configuration. Remote commands are executed using the delegated credentials of whatever user invoked the commands, meaning they can only do what they have permission to do - and what they could presumably do through a half-dozen other tools anyway. Whatever auditing you have in place in your environment cannot be bypassed by Remoting. Unlike many past "remote execution" solutions, Remoting does not operate under a single "super-privileged" account unless you expressly configure it that way (which requires several steps and cannot possibly by accomplished accidentally, as it requires the creation of custom endpoints).

Remember: Anything someone can do via Remoting, they can already do in a half-dozen other ways. Remoting simply provides a more consistent, controllable, and scalable means of doing so.

## Remoting is Lower Overhead

Unlike Remote Desktop Connection (RDC, which many Administrators currently use to manage remote servers), Remoting is very low-overhead. It does not require the server to spin up an entire graphical operating environment, impacting server performance and memory management. Remoting is also more scalable, enabling authorized users (mainly Administrators in most cases) to execute commands against multiple servers at once - which improves consistency and reduces error, while also speeding up response times and lowering administrative overhead.

Remoting is Microsoft's way forward. To not use Remoting is to deliberately attempt to use Windows in a way that it was explicitly designed not to do. You will reduce, not improve your security, while also increasing operational overhead, enabling greater instance of human error, and reducing server performance. Microsoft Administrators have for decades been toiling under an operational paradigm that was wrong-headed and short-sighted; Remoting is finally delivering to Windows the administrative model that every other network operating system has used for years, if not decades.

## Remoting Uses Mutual Authentication

Unlike nearly every other remote management technique out there - including tools like PSExec and even, under some circumstances, Remote Desktop, PowerShell Remoting by default requires mutual authentication. The user attempting to connect to a server is authenticated and known; the system also ensures that the server connected to is the intended server and not an imposter. This provides far better security than past techniques, while also helping to reduce error - you can't "accidentally log on to the wrong console" as you could if you just walked into the data center.

## Summary

At this point, denying PowerShell Remoting is like denying Ethernet: It's ridiculous to think you'll successfully operate your environment without it. For the first time, Microsoft has provided a supported, official, baked-in technology for remote server administration that does not use elevated credentials, does not store credentials in any way, that supports mutual authentication, and that is complete security-transparent. This is the administration technology we should have had all along; moving to it will only make your environment more manageable and more secure, not less.

