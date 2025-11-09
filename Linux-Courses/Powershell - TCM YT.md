## Cmdlets
Verb-Noun
action-target of that action
==ex: Get-Process; Stop-Service; Set-Date; Get-Date

Get-Command : list all commands , version, source name
Get-Help - Built in Manual system

## Alisases
Get-ChildItem - gci

```Powershell
Get-Alias
```
Gets all the Aliases in our system

```
Get-Alias -Definition Get-ChildItem
```

This is a shorcut to get the Aliases for a command we are looking for.

Invoke_WebRequest is used to send HTTP or HTTPS requests ot a webserver & retrieve the response.
```Get-Alias -Definition Invoke-WebRequest
```
![[Pasted image 20251102114248.png]]

Invoke-Expression takes a string of text & executes it as a command or script, allowing dynamic code execution at runtime.

![[Pasted image 20251102114436.png]]

![[Pasted image 20251102115952.png]]

![[Pasted image 20251102120526.png]]

 ![[Pasted image 20251102120650.png]]
```
Get-Process | Get-Member
```
The command `Get-Process | Get-Member` combines two key PowerShell commandlets using the **pipeline** operator (`|`).

Here's what each part does:

- **`Get-Process`**: This commandlet retrieves information about all running processes on your system (3:11). Unlike traditional command-line tools that output plain text, PowerShell commandlets like `Get-Process` output **objects**. Each object represents a running process and contains various properties (like name, ID, CPU usage) and methods (actions you can perform on it) (11:10-11:29).
    
- **`|` (Pipeline operator)**: This operator takes the output of the command on its left (`Get-Process`'s objects) and passes it as input to the command on its right (`Get-Member`) (12:20-12:30).
    
- **`Get-Member`**: This is a crucial commandlet for understanding PowerShell objects (11:39). When `Get-Member` receives the process objects from `Get-Process` via the pipeline, it inspects them and displays all the **properties** (data attributes) and **methods** (actions) associated with those objects (11:44-11:54).
    

So, `Get-Process | Get-Member` essentially tells you: "Show me all the running processes, and then for each of those process objects, tell me what kind of information (properties) they contain and what actions (methods) I can perform on them."

![[Pasted image 20251102121058.png]]

### What is an object in Powershell
In PowerShell, an **object** is a structured piece of data that represents an item or entity. Unlike traditional command-line environments that primarily output plain text, PowerShell focuses on passing these rich, structured objects between commands (10:27-11:04).

Think of an object like a detailed record or a blueprint for something. For example, when you run `Get-Process`, PowerShell doesn't just give you a wall of text; it gives you a collection of "process objects." Each of these objects contains:

- **Properties**: These are the attributes or characteristics of the object. For a process object, properties include things like its **Name**, **ID**, **CPU** usage, and **Start Time** (11:16-11:25).
- **Methods**: These are the actions or functions you can perform on that object. For a process object, a method might be `Kill` (to terminate the process) or `Start` (if it's a process that can be started) (11:26-12:11).

This object-oriented approach makes PowerShell much more powerful and flexible, as you can easily access, manipulate, and filter specific pieces of data from these objects using commandlets like `Where-Object` or `Select-Object`, without needing to parse text (14:40-14:46).

### Properties in Powershell
n PowerShell, a property is a specific piece of information or an attribute that describes an object. Think of it as a field in a database record or a characteristic that belongs to something.

For example, when Get-Process returns a collection of process objects, each process object has various properties such as:

Name: The name of the process executable (16:27-16:29).
ID: The unique Process ID (PID) (16:23-16:25).
CPU: The total processor time used by the process (16:18-16:20).
Handles: The number of object handles the process has open (16:07-16:09).
StartType: For a service object, this property indicates if it starts automatically (12:50-12:52).
The video demonstrates how you can work with these properties:

You can see all available properties for an object using Get-Member (11:58-12:03).
You can filter objects based on their property values using Where-Object. For instance, the video shows filtering services where the StartType property is 'Automatic' (12:50-12:52, 14:09-14:12).
You can select specific properties to display using Select-Object. The video demonstrates selecting only the Name and ID properties from process objects (16:37-16:40).
You can also access a property of a stored object using dot notation, like $product.ProductName to get just the product name from the registry object (24:07-24:10).
Properties are fundamental to working with PowerShell objects because they allow you to access and manipulate the specific data points that define an object.

### Methods in Powershell
In PowerShell, a **method** is an action or a function that an object can perform. While properties describe an object (what it _is_), methods describe what an object _can do_.

Think of it this way: if a process object has properties like its `Name` or `ID`, it might have methods like `Kill` or `Stop`. The video mentions the `Kill` method for a process object, which would be used to terminate that specific process (11:26-11:30, 12:08-12:10).

You can discover the methods available for any object by piping it to `Get-Member`, just like you would for properties. For example, `Get-Process | Get-Member` will list both the properties and methods associated with process objects (11:51-11:54).

Methods allow you to interact with and control the objects directly within PowerShell, performing specific operations on them.

### Using a method in powershell
To use a method on an object in PowerShell, you generally follow this syntax:

$VariableName.MethodName()

Or, if the object is the direct output of a command:

(CommandThatOutputsObject).MethodName()

Here's a breakdown and an example:

Get the object: First, you need an object to work with. You can get an object by running a commandlet (like Get-Process) and often storing its output in a variable.
Dot notation: You then use a period (.) after the object (or the variable holding the object) to access its properties and methods.
Method name: After the period, you type the name of the method you want to use.
Parentheses (): Most methods require parentheses after their name, even if they don't take any arguments. If a method requires arguments, you would put them inside the parentheses.
Example using the Kill method from the video (11:26-11:30):

Let's say you want to stop a specific process, for instance, a process named "notepad".

You would first get the Notepad process object:
$notepadProcess = Get-Process -Name notepad

Then, to use the Kill method on that specific $notepadProcess object, you would type:
$notepadProcess.Kill()

This command tells PowerShell to execute the Kill method on the $notepadProcess object, which would result in the Notepad application being terminated.


```
Get-Service | Where-Object { $_.StartType -eq 'Automatic'}
```
This command is a great example of how PowerShell uses **objects** and the **pipeline** to filter data effectively, as explained in the video starting around 12:36.

Let's break it down:

- **`Get-Service`**: This commandlet retrieves all services currently installed on your system. Importantly, it outputs these services as **objects**, not just plain text (12:48-13:10). Each service object contains various properties, such as its name, status, and what the video refers to as its `StartType` (12:55-13:02).
    
- **`|` (Pipeline operator)**: This operator takes the output of `Get-Service` (all the service objects) and passes them one by one as input to the `Where-Object` commandlet on the right (13:04-13:16).
    
- **`Where-Object { $_.StartType -eq 'Automatic' }`**: This is the filtering part of the command.
    
    - **`Where-Object`**: This commandlet is designed to filter objects based on a condition (13:20-13:28).
    - **`{ ... }` (Script Block)**: The curly braces define a "script block," which is a chunk of PowerShell code that `Where-Object` executes for _each_ object it receives from the pipeline (13:28-13:38).
    - **`$_` (Current Object Variable)**: Inside the script block, `$_` (or `$PSItem`) is a special automatic variable that represents the _current_ object being processed from the pipeline (13:47-13:56). So, as each service object comes through, `$_` temporarily holds that specific service.
    - **`.StartType`**: This accesses the `StartType` property of the current service object (14:09-14:12).
    - **`-eq 'Automatic'`**: This is a comparison operator that checks if the `StartType` property of the current service object is **equal to** the string 'Automatic' (14:13-14:23).

**In summary**: The entire command tells PowerShell to "Get all system services, then for each service, check if its 'StartType' property is set to 'Automatic'. If it is, include that service in the final output; otherwise, skip it." (14:24-14:36) This allows you to quickly find only the services configured to start automatically.