# Windows - DISM

```powershell title="Clean Up Component Store (Windows update files)"
dism.exe /online /cleanup-Image /AnalyzeComponentStore
dism.exe /online /cleanup-Image /StartComponentCleanup
```
