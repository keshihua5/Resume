# How To Fix Offboarding Error: Target user \'xxx\' already has a primary mailbox

Monday, July 20, 2020

2:04 PM

------

## Fix For Large Group Of Users: 
Do the following steps:
> Note This example is for a carentan machine\.

1. Connect the on-prem machine.

2. Get on-prem mailbox.

   Open a Powershell window as Admin, and run the following:

   `cd C:\\Users\\exo\\Desktop`

   `. .\\dotsource-exchangeshell.ps1 -onpremsession`

   `\$gmb = get-mailbox -Resultsize Unlimited \| ?{\$\_.Name -match \'ctantest\'}`

   `\$onpremMB = \$gmb \| ?{\$\_.Name -match \'ctantest\'} \| Select-Object -Property @{Name=\"onpremMB\";Expression = {\$\_.Name}}`

   `\$onpremMB \| export-clixml CTAN_onpremMB_0720.xml`

3. Get cloud mailbox and find the dupe.

   Open another Powershell window as Admin, and run the following:

    `cd C:\\Users\\exo\\Desktop`

    `. .\\dotsource-exchangeshell.ps1 -onlinesession`

   `\$gmb = get-mailbox -Resultsize Unlimited \| ?{\$\_.Name -match \'ctantest\'}`

   `\$onpremMB = import-clixml CTAN_onpremMB_0720.xml` 

   `\$dupes = \$gmb \| ?{\$\_.Name -in \$onpremMB.onpremMB}` 

   `\$dupes \| Select-Object -Property @{Name=\"dupeMB\";Expression = {\$\_.Name}} \|`

   `export-clixml CTAN_dupeMB_0720.xml` 

   `for (\$i=0; \$i -lt \$dupes.Count; \$i++)`

   `{`

   `\$upn = \$dupes\[\$i\].Name + \"\@carentancasino.com\"`

   `Remove-MsolUser -UserPrincipalName \$upn -Force:\$true`

   `Remove-MsolUser -UserPrincipalName \$upn -RemoveFromRecycleBin -Force:\$true`

   `}`

   `Start-ADSyncSyncCycle`

4. To check, run the following (it will take approximately 5-10 minutes):

   `\$gmb = get-mailbox -Resultsize Unlimited \| ?{\$\_.Name -match \'ctantest\'}`

   `\$dupes = \$gmb \| ?{\$\_.Name -in \$onpremMB.onpremMB}` 

   `\$dupes.Count`

   This should be 0.

   If it is not gone by then, run the `Sync` call again; wait and check. 

   Or run  `remove-msoluser` again.

 
