# Ashift Tool
* you must have `Ashift` tool to use this skile

## Download 
> * **NOTE:** The download link might be somewhat outdated, but you can download the version you want from this link: [TOOL PAGE](https://github.com/saa-999/Ashift)
### Quick Download
**Linux & Mac** `sudo curl -L -o /usr/local/bin/Ashift https://github.com/saa-999/Ashift/releases/download/v1.1.0/Ashift && sudo chmod +x /usr/local/bin/Ashift` **Version 1.1.0**

**Windows (PowerShell):**

Assuming you have a `C:\Tools` directory added to your System PATH:
`Invoke-WebRequest -Uri "https://github.com/saa-999/Ashift/releases/download/v1.1.0/Ashift.exe" -OutFile "C:\Tools\Ashift.exe"`
The tool will not work just anywhere in the Windows command line; you must execute the command within the environment variable:
`[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Tools", "User")`
 