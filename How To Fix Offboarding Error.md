# How To Fix Offboarding Error: Target user \'xxx\' already has a primary mailbox

Monday, July 20, 2020

2:04 PM

------

**Contents**

[Fix For Single Users](#fix-for-single-users)

[Fix For Large Group Of Users](#fix-for-large-group-of-users)

------

## Fix For Single Users: 

**Reference:**  
[Offboarding Error: Target user nnnUser already has a primary mailbox](https://microsoft.sharepoint.com/teams/SRELivesite/Shared%20Documents/%20Migration%20Health%20Infra/Migration%20Health%20Infra/Failure%20Management%20-%20SOPs.one#Offboarding%20Error%20Target%20user%20%27nnnUser%27%20already%20has%20a%20primary&section-id=%7B596DF710-FE29-4DC7-BBF7-874255F6E1B0%7D&page-id=%7BF275AB93-0C47-4B8A-BB2E-2619425BB2D6%7D&end)  
([Web view](https://microsoft.sharepoint.com/teams/SRELivesite/_layouts/OneNote.aspx?id=/teams/SRELivesite/Shared%20Documents/%20Migration%20Health%20Infra/Migration%20Health%20Infra&wd=target%28Failure%20Management%20-%20SOPs.one%7c596DF710-FE29-4DC7-BBF7-874255F6E1B0/Offboarding%20Error:%20Target%20user%20%27nnnUser%27%20already%20has%20a%20primary%7cF275AB93-0C47-4B8A-BB2E-2619425BB2D6/%29))

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

 
