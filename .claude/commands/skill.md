# Crop Card Images

Crop downloaded square card images into the correct dimensions for TCG Arena.

## Usage

When asked to crop images from a source folder into `Images/`, run the PowerShell script below.

**Portrait:** 616×850, origin **(233, 160)**  
**Landscape:** 850×616, origin **(110, 260)**  
Current landscape exceptions: `B10019, B10082, B10083, B10100, B10101, B10102`

## Script

```powershell
Add-Type -AssemblyName System.Drawing

$src = "F:\Conan\x"   # folder with downloaded 1080x1080 images
$dst = "F:\projects\ConanTCG\TCG-Arena-DetectiveConan\Images"
$landscape = @('B10019','B10082','B10083','B10100','B10101','B10102')

$enc = [System.Drawing.Imaging.ImageCodecInfo]::GetImageEncoders() |
       Where-Object { $_.MimeType -eq 'image/jpeg' } | Select-Object -First 1
$p = New-Object System.Drawing.Imaging.EncoderParameters(1)
$p.Param[0] = New-Object System.Drawing.Imaging.EncoderParameter(
                  [System.Drawing.Imaging.Encoder]::Quality, 92L)

$count = 0; $errors = @()

foreach ($f in Get-ChildItem $src -Filter "*.jpg") {
    $base = [System.IO.Path]::GetFileNameWithoutExtension($f.Name)
    $isLandscape = $landscape -contains $base

    # Portrait: (233,160) 616x850  |  Landscape: (110,260) 850x616
    if ($isLandscape) { $ox=110; $oy=260; $cw=850; $ch=616 }
    else              { $ox=233; $oy=160; $cw=616; $ch=850 }

    try {
        $img = [System.Drawing.Bitmap]::new($f.FullName)
        $bmp = New-Object System.Drawing.Bitmap($cw, $ch)
        $g   = [System.Drawing.Graphics]::FromImage($bmp)
        $g.DrawImage($img,
            [System.Drawing.Rectangle]::new(0, 0, $cw, $ch),
            [System.Drawing.Rectangle]::new($ox, $oy, $cw, $ch),
            [System.Drawing.GraphicsUnit]::Pixel)
        $g.Dispose(); $img.Dispose()
        $bmp.Save((Join-Path $dst $f.Name), $enc, $p)
        $bmp.Dispose()
        $count++
    } catch { $errors += "$($f.Name): $_" }
}

"Done: $count images cropped"
if ($errors) { "Errors:"; $errors }
```

## Notes

- Source images must be 1080×1080 squares.
- To add more landscape exceptions, add the base filename to `$landscape`.
- Quality is saved at 92 (JPEG).
