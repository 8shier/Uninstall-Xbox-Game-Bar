# Uninstall-Xbox-Game-Bar
完全卸载Xbox Game Bar

Completely uninstall Xbox Game Bar

<span style="font-size:18px; font-weight:bold; color:red;">1️⃣</span>首先在任务管理器中终止关于Xbox Game Bar的所有进程

<span style="font-size:18px; font-weight:bold; color:red;">2️⃣</span>按win+r输入Powershell并确认

<span style="font-size:18px; font-weight:bold; color:red;">3️⃣</span>粘贴以下代码并执行
```powershell
Get-AppxPackage Microsoft.XboxGamingOverlay | Remove-AppxPackage
```
这里Xbox Game Bar组件已经基本移除，但是会不时弹出 “ms-gamingoverlay-link” 的弹窗报错

<span style="font-size:18px; font-weight:bold; color:red;">4️⃣</span>这时候新建一个文本文件并用记事本打开，粘贴以下代码
```powershell
@(set ^ "0=%~f0" -des ') &set 1=%*& powershell -nop -c iex(out-string -i (gc -lit $env:0)) & exit /b ')
# AveYo: fix annoyance after uninstalling Xbox, AveYo 2024.12.27

$n0 = 'ms-gamebar-annoyance'
$s0 = 'active'
if (gp Registry::HKCR\ms-gamebar NoOpenWith -ea 0) { $s0 = 'inactive' }

#:: Args / Dialog - to skip the prompt can use commandline parameters or rename script: ms-gamebar-annoyance disable.bat
$do = ''; $cl = @{0 = 'enable'; 1 = 'disable'; 2 = 'cancel'} ; if (!$env:0) {$env:0 = "$pwd\.pasted"} 
foreach ($a in $cl.Values) {if ("$(split-path $env:0 -leaf) $env:1" -like "*$a*") {$do = $a} }
if ($do -eq '') {
  $choice = (new-object -ComObject Wscript.Shell).Popup("state: $s0  -  No to disable", 0, $n0, 0x1043)
  if ($choice -eq 2) {$do = $cl[2]} elseif ($choice -eq 7) {$do = $cl[1]} else {$do = $cl[0]} ; $env:1 = $do
  if ($do -eq 'cancel') {return}
}

$toggle = (0,1)[$do -eq 'enable']
sp "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\GameDVR" "AppCaptureEnabled" $toggle -type dword -force -ea 0
sp "HKCU:\System\GameConfigStore" "GameDVR_Enabled" $toggle -type dword -force -ea 0

$cc = { 
  [Console]::Title = "$($args[2]) $($args[1])"
  $toggle = (0,1)[($args[1]) -eq 'enable']
  sp "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\GameDVR" "AppCaptureEnabled" $toggle -type dword -force -ea 0
  sp "HKCU:\System\GameConfigStore" "GameDVR_Enabled" $toggle -type dword -force -ea 0
  "ms-gamebar","ms-gamebarservices","ms-gamingoverlay" |foreach {
    if (!(test-path "Registry::HKCR\$_\shell")) {ni "Registry::HKCR\$_\shell" -force >''}
    if (!(test-path "Registry::HKCR\$_\shell\open")) {ni "Registry::HKCR\$_\shell\open" -force >''}
    if (!(test-path "Registry::HKCR\$_\shell\open\command")) {ni "Registry::HKCR\$_\shell\open\command" -force}
    sp "Registry::HKCR\$_" "(Default)" "URL:$_" -force
    sp "Registry::HKCR\$_" "URL Protocol" "" -force
    if ($toggle -eq 0) {
      sp "Registry::HKCR\$_" "NoOpenWith" "" -force
      sp "Registry::HKCR\$_\shell\open\command" "(Default)" "`"$env:SystemRoot\System32\systray.exe`"" -force
    } else {
      rp "Registry::HKCR\$_" "NoOpenWith" -force -ea 0
      ri "Registry::HKCR\$_\shell" -rec -force -ea 0 
    }
  }
  start ms-gamebar://annoyance # AveYo: test if working
}

if ([Security.Principal.WindowsIdentity]::GetCurrent().Groups.Value -notcontains 'S-1-5-32-544') {
  write-host " Requesting ADMIN rights.. " -fore Black -back Yellow; sleep 2 
  sp HKCU:\Volatile*\* $n0 ".{$cc} '$($env:0-replace"'","''")' '$($env:1-replace"'","''")' '$n0'" -force -ea 0
  start powershell  -args "-nop -c iex(gp Registry::HKU\S-1-5-21*\Volatile*\* '$n0' -ea 0).'$n0'" -verb runas
} else {. $cc "$env:0" "$env:1" "$n0" }    

$Press_Enter_if_pasted_in_powershell
```
<span style="font-size:18px; font-weight:bold; color:red;">5️⃣</span>Ctrl+s保存文件并关闭，右键文件重命名，将文件后缀名txt改为bat

<span style="font-size:18px; font-weight:bold; color:red;">6️⃣</span>双击执行该文件，然后会弹出一个窗口，点击否

到这里就完全卸载了烦人的Xbox Game Bar并不再弹窗
