# Caldera Ability: Get Chrome Extensions

## Ability Details

- **Tactic:** Discovery
- **Technique Name:** Browser Extensions
- **Technique ID:** T1176
- **Ability Name:** `[custom] Get Chrome Extensions`
- **Description:** Extracts the names of installed Chrome extensions for the current user by analyzing `manifest.json` files in the Chrome extensions directory.

## Execution

The ability executes the following PowerShell command:

```powershell
$extensionsPath = "C:\Users\$((Get-WmiObject -Class Win32_ComputerSystem).UserName.Split('\')[1])\AppData\Local\Google\Chrome\User Data\Default\Extensions";
$extensions = Get-ChildItem -Path $extensionsPath;
$extensions | ForEach-Object {
    $manifestPath = "$($_.FullName)\$($_.GetDirectories()[0].Name)\manifest.json";
    if (Test-Path $manifestPath) {
        $manifest = Get-Content $manifestPath | ConvertFrom-Json;
        $name = $manifest.name;
        if ($name -like "__MSG_*") {
            $localePath = "$($_.FullName)\$($_.GetDirectories()[0].Name)\_locales\en\messages.json";
            if (Test-Path $localePath) {
                $messages = Get-Content $localePath | ConvertFrom-Json;
                $key = $name -replace "__MSG_", "" -replace "__", "";
                $name = $messages.$key.message;
            }
        }
        [PSCustomObject]@{ ID = $_.Name; Name = $name }
    }
} | Format-Table -AutoSize
```

## Features

- Retrieves Chrome extension names for the current user.
- Resolves localized extension names if applicable.
- Formats the output in a table for readability.

## Requirements

- Windows OS
- PowerShell
- Chrome installed with user extensions

## Usage

To use this ability in Caldera, upload the YAML file to your Caldera instance and assign it to an appropriate adversary profile under the Discovery tactic.

## License

This project is open-source. Feel free to modify and improve it as needed.

## Contributing

Contributions are welcome! If you find issues or want to add improvements, feel free to submit a pull request.

## Author

[Your Name or GitHub Handle]

