# Windows

## Remove/Delete ExcludedPortRange by Hyper-V or something

<!--- thank you Hyper-V for reserving thousands of ports on my computer takes me hours to figure out why some of my services are unable to start nearly every time I install an update YOU FUCKING IDIOTS! -->

Open `cmd.exe` in elevated mode, and type:

```bat
start /wait "" reg add HKLM\SYSTEM\CurrentControlSet\Services\hns\State /v EnableExcludedPortRange /d 0 /f
```

Source: https://dandini.wordpress.com/2019/07/15/administered-port-exclusions-blocking-high-ports/

