# Hordozható Viber-artifact ZIP

A korábbi Windows PowerShell Compress-Archive visszaperjeles ZIP-bejegyzéseket készített.
Az archívum tartalma helyes volt, de Unix unzip figyelmeztetést és 1-es kódot adhatott.
Az eredeti bizonyítékcsomagot nem írjuk felül; a hordozható másolat csak a bejegyzések
útvonal-elválasztóját normalizálja `/` jelre, a fájlok byte-jai és a belső SHA256SUMS változatlanok.
A `viber-speech-act-20261107-100156-portable.zip` mind a 27 hash-e helyes, és `unzip -tq`
0-s kóddal, figyelmeztetés nélkül ellenőrizte.

Jövőbeli Windows-futtatásoknál a végső Compress-Archive helyett ez a beépített .NET
csomagolás használható, új függőség nélkül. Az `$artifacts` és `$run` a mérés változói.
A kód új célfájlt kér, meglevő bizonyítékot nem ír felül. Ez csomagolás, nem modellfuttatás.

```powershell
Add-Type -AssemblyName System.IO.Compression
Add-Type -AssemblyName System.IO.Compression.FileSystem
$root = (Resolve-Path -LiteralPath $artifacts).Path
$zipPath = Join-Path (Get-Location).Path "$run.zip"
$stream = [System.IO.File]::Open($zipPath, [System.IO.FileMode]::CreateNew)
$archive = $null
try {
    $archive = [System.IO.Compression.ZipArchive]::new(
        $stream, [System.IO.Compression.ZipArchiveMode]::Create)
    Get-ChildItem -LiteralPath $root -Recurse -File | Sort-Object FullName | ForEach-Object {
        $entry = $_.FullName.Substring($root.Length + 1).Replace('\', '/')
        [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile(
            $archive, $_.FullName, $entry,
            [System.IO.Compression.CompressionLevel]::Optimal) | Out-Null
    }
}
finally {
    if ($archive) { $archive.Dispose() }
    $stream.Dispose()
}
```

A fenti PowerShell-részletet ebben a Linux-környezetben nem futtattuk. A mellékelt
hordozható ZIP a tényleges eredeti csomagból készült; byte- és hash-ellenőrzése, valamint
Unix unzip-ellenőrzése megtörtént.
