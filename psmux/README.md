심볼릭 링크로 적용 (PowerShell, 개발자 모드 필요):
```powershell
New-Item -ItemType SymbolicLink -Path $HOME\.psmux.conf -Target C:\Projects\environments\psmux\.psmux.conf
```

설정 다시 읽기 (psmux 안에서):
```
prefix r
```
