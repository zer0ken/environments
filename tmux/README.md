tmux / psmux 공용 설정.

## 심볼릭 링크로 적용

WSL/Linux (tmux):
```bash
ln -s ~/projects/environments/tmux/.tmux.conf ~/.tmux.conf
```

Windows (psmux, PowerShell · 개발자 모드 필요):
```powershell
New-Item -ItemType SymbolicLink -Path $HOME\.tmux.conf -Target C:\Projects\environments\tmux\.tmux.conf
```

## 설정 다시 읽기 (안에서)
```
prefix r
```
