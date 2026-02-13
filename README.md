# Windows Temp Temizleme Aracı

Windows 11 için günlük Temp klasörü temizleme aracı.

## Dosya Listesi

| Dosya | Açıklama |
|-------|----------|
| `Clean-WindowsTemp.ps1` | Ana temizleme script'i |
| `WindowsTempCleanup-Task.xml` | Task Scheduler yapılandırması |
| `Install-WindowsTempCleanup.ps1` | Kurulum script'i |
| `Uninstall-WindowsTempCleanup.ps1` | Kaldırma script'i |

## Kurulum

### Yöntem 1: Otomatik Kurulum (Önerilen)

1. Tüm dosyaları bir klasöre kaydedin (örn: `C:\Temp\WindowsTempCleanup\`)

2. **PowerShell'i Administrator olarak açın:**
   - Başlat menüsüne `powershell` yazın
   - Sağ tık → **"Run as administrator"**

3. Kurulum script'ini çalıştırın:
   ```powershell
   cd C:\Temp\WindowsTempCleanup
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
   .\Install-WindowsTempCleanup.ps1
   ```

### Yöntem 2: Manuel Kurulum

1. Script'i program dosyalarına kopyalayın:
   ```powershell
   New-Item -Path "C:\Program Files\WindowsTempCleanup" -ItemType Directory -Force
   Copy-Item "Clean-WindowsTemp.ps1" "C:\Program Files\WindowsTempCleanup\"
   ```

2. Task Scheduler görevini oluşturun:
   ```powershell
   $action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File `"C:\Program Files\WindowsTempCleanup\Clean-WindowsTemp.ps1`" -IncludeUserTemp"
   $trigger = New-ScheduledTaskTrigger -Daily -At 3am
   $settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopIfGoingOnBatteries
   $principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
   Register-ScheduledTask -TaskName "WindowsTempCleanup" -Action $action -Trigger $trigger -Settings $settings -Principal $principal -Force
   ```

## Kullanım

### Komut Satırı Seçenekleri

| Komut | Açıklama |
|-------|----------|
| `.\Clean-WindowsTemp.ps1` | Normal temizleme (Administrator gerekli) |
| `.\Clean-WindowsTemp.ps1 -ShowStatus` | Sadece durum göster |
| `.\Clean-WindowsTemp.ps1 -DryRun` | Önizleme modu (silmez) |
| `.\Clean-WindowsTemp.ps1 -IncludeUserTemp` | Kullanıcı temp'ini de temizle |
| `.\Clean-WindowsTemp.ps1 -Force` | Kullanımdaki dosyaları da silmeye çalış |

### Örnekler

```powershell
# Durumu görüntüle
.\Clean-WindowsTemp.ps1 -ShowStatus

# Önizleme yap (hiçbir şey silmez)
.\Clean-WindowsTemp.ps1 -DryRun -IncludeUserTemp

# Tam temizleme
.\Clean-WindowsTemp.ps1 -IncludeUserTemp
```

## Zamanlama

Task Scheduler görevi şu şekilde yapılandırılmıştır:

- **Çalışma zamanı:** Her gün saat 03:00
- **Yetki:** SYSTEM hesabı (en yüksek yetki)
- **Zaman aşımı:** 1 saat
- **Öncelik:** Düşük (sistemi yormaz)

## 🔧 Yönetim

### Görevi Manuel Çalıştırma

```powershell
Start-ScheduledTask -TaskName "WindowsTempCleanup"
```

### Görevi Durdurma

```powershell
Stop-ScheduledTask -TaskName "WindowsTempCleanup"
```

### Görev Durumunu Görme

```powershell
Get-ScheduledTask -TaskName "WindowsTempCleanup" | Format-List *
```

### Log Dosyasını Görüntüleme

```powershell
Get-Content "C:\ProgramData\WindowsTempCleanup\cleanup.log" -Tail 50
# veya
notepad "C:\ProgramData\WindowsTempCleanup\cleanup.log"
```

## Kaldırma

```powershell
.\Uninstall-WindowsTempCleanup.ps1
```

Bu komut:
- Task Scheduler görevini kaldırır
- Script dosyalarını siler
- İsterseniz log dosyalarını da temizler

## Temizlenen Dizinler

| Dizin | Açıklama |
|-------|----------|
| `C:\Windows\Temp` | Windows sistem temp dosyaları |
| `%TEMP%` | Kullanıcı temp dosyaları (`-IncludeUserTemp` ile) |

## Önemli Notlar

1. **Administrator yetkisi gereklidir** - `C:\Windows\Temp` için
2. **Kullanımdaki dosyalar atlanır** - Hata önlemek için
3. **İlk çalıştırmada** `-DryRun` ile önizleme yapmanız önerilir
4. **Log dosyası** tüm işlemleri kaydeder

## Güvenlik

- Script sadece temp dosyalarını siler
- Sistem dosyalarına dokunmaz
- Kullanımdaki dosyaları atlar
- Tüm işlemler loglanır
