Claude Code 전역 규칙 (`~/.claude/CLAUDE.md`).

## 심볼릭 링크로 적용

Windows (PowerShell · 개발자 모드 필요):
```powershell
New-Item -ItemType SymbolicLink -Path $HOME\.claude\CLAUDE.md -Target C:\Projects\environments\claude\CLAUDE.md
```

WSL/Linux:
```bash
ln -s ~/projects/environments/claude/CLAUDE.md ~/.claude/CLAUDE.md
```
