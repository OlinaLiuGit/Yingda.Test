﻿# Complete-Transaction

## SYNOPSIS
Commits the active transaction.

## DESCRIPTION
The Complete-Transaction cmdlet commits an active transaction.
When you commit a transaction, the commands in the transaction are finalized and the data affected by the commands is changed.

If the transaction includes multiple subscribers, to commit the transaction, you must enter one Complete-Transaction command for every Start-Transaction command.

The Complete-Transaction cmdlet is one of a set of cmdlets that support the transactions feature in Windows PowerShell.
For more information, see about_Transactions.

## PARAMETERS

### InformationAction [System.Management.Automation.ActionPreference]

```powershell
[Parameter(ParameterSetName = 'Set 1')]
[ValidateSet(
  'SilentlyContinue',
  'Stop',
  'Continue',
  'Inquire',
  'Ignore',
  'Suspend')]
```




### InformationVariable [System.String]

```powershell
[Parameter(ParameterSetName = 'Set 1')]
```




### Confirm [switch]

```powershell
[Parameter(ParameterSetName = 'Set 1')]
```

Prompts you for confirmation before running the cmdlet.Prompts you for confirmation before running the cmdlet.


### WhatIf [switch]

```powershell
[Parameter(ParameterSetName = 'Set 1')]
```

Shows what would happen if the cmdlet runs.
The cmdlet is not run.Shows what would happen if the cmdlet runs.
The cmdlet is not run.



## INPUTS
### None

You cannot pipe objects to Complete-Transaction.

## OUTPUTS
### None

This cmdlet does not return any objects.

## NOTES
You cannot roll back a transaction that has been committed, or commit a transaction that has been rolled back.

You cannot roll back any transaction other than the active transaction.
To roll back a different transaction, you must first commit or roll back the active transaction.

By default, if any part of a transaction cannot be committed, such as when a command in the transaction results in an error, the entire transaction is rolled back.


## EXAMPLES
### -------------------------- EXAMPLE 1 --------------------------

```powershell
PS C:\>cd hkcu:\software
PS HKCU:\software> start-transaction
PS HKCU:\software> new-item MyCompany -UseTransaction
PS HKCU:\software> dir m*
Hive: HKEY_CURRENT_USER\software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}

PS HKCU:\software> complete-transaction
PS HKCU:\software> dir m*
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}
0   0 MyCompany                      {}

```
This example shows the effect of using the Complete-Transaction cmdlet to commit a transaction.

The Start-Transaction command starts the transaction.
The New-Item command uses the UseTransaction parameter to include the command in the transaction.

The first "dir" (Get-ChildItem) command shows that the new item has not yet been added to the registry.

The Complete-Transaction command commits the transaction, which makes the registry change effective.
As a result, the second "dir" command shows that the registry is changed.







### -------------------------- EXAMPLE 2 --------------------------

```powershell
PS C:\>cd hkcu:\software
PS HKCU:\software> start-transaction
PS HKCU:\software> new-item MyCompany -UseTransaction
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
0   0 MyCompany                      {}

PS HKCU:\software> start-transaction
PS HKCU:\Software> Get-Transaction

RollbackPreference   SubscriberCount  Status
------------------   ---------------  ------
Error                2                Active

PS HKCU:\software> new-itemproperty -path MyCompany -name MyKey -value -UseTransaction

MyKey
-----
123

PS HKCU:\software> complete-transaction
PS HKCU:\software> get-transaction

RollbackPreference   SubscriberCount  Status
------------------   ---------------  ------
Error                1                Active

PS HKCU:\software> dir m*
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}

PS HKCU:\software> complete-transaction
PS HKCU:\software> dir m*
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}
0   1 MyCompany                      {MyKey}

```
This example shows how to use Complete-Transaction to commit a transaction that has more than one subscriber.

To commit a multi-subscriber transaction, you must enter one Complete-Transaction command for every Start-Transaction command.
The data is changed only when the final Complete-Transaction command is submitted.

For demonstration purposes, this example shows a series of commands entered at the command line.
In practice, transactions are likely to be run in scripts, with the secondary transaction being run by a function or helper script that is called by the main script.

In this example, a Start-Transaction command starts the transaction.
A New-Item command with the UseTransaction parameter adds the MyCompany key to the Software key.
Although the New-Item command returns a key object, the data in the registry is not yet changed.

A second Start-Transaction command adds a second subscriber to the existing transaction.
The Get-Transaction command confirms that the subscriber count is 2.
A New-ItemProperty command with the UseTransaction parameter adds a registry entry to the new MyCompany key.
Again, the command returns a value, but the registry is not changed.

The first Complete-Transaction command reduces the subscriber count by 1.
This is confirmed by a Get-Transaction command.
However, no data is changed, as evidenced by a "dir m*" (Get-ChildItem) command.

The second Complete-Transaction command commits the entire transaction and changes the data in the registry.
This is confirmed by a second "dir m*" command, which shows the changes.






### -------------------------- EXAMPLE 3 --------------------------

```powershell
PS C:\>cd hkcu:\software
PS HKCU:\software> start-transaction
PS HKCU:\software> new-item MyCompany -UseTransaction
PS HKCU:\software> dir m*
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}

PS HKCU:\software> dir m* -UseTransaction
Hive: HKEY_CURRENT_USER\Software

SKC  VC Name                           Property
---  -- ----                           --------
82   1 Microsoft                      {(default)}
0   0 MyCompany                      {}

PS HKCU:\software> complete-transaction

```
This example shows the value of using Get-* commands, and other commands that do not change data, in a transaction.
When a Get-* command is used in a transaction, it gets the objects that are part of the transaction.
This allows you to preview the changes in the transaction before the changes are committed.

In this example, a transaction is started.
A New-Item command with the UseTransaction parameter adds a new key to the registry as part of the transaction.

Because the new registry key is not added to the registry until the Complete-Transaction command is run, a simple "dir" (Get-ChildItem) command shows the registry without the new key.

However, when you add the UseTransaction parameter to the "dir" command, the command becomes part of the transaction, and it gets the items in the transaction even if they are not yet added to the data.







## RELATED LINKS

[Online Version:](http://go.microsoft.com/fwlink/p/?linkid=289802)

[Get-Transaction]()

[Start-Transaction]()

[Undo-Transaction]()

[Use-Transaction]()

[about_Transactions]()
