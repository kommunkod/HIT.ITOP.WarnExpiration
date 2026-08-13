# iTop Warn Expiration
Trigger class to check if a date on a given object is approaching expiration. Adds an additional Trigger to the Notification flow and runs with cron every few hours. Notifies once per object on the following conditions:

- An object of type T (iTop Class, setting per trigger) with the field E (Date or DateTime field, setting per trigger) passes N (Amount of days before date passes, setting per trigger)

- There has been no notification sent for this trigger between now and the date in field E minus N days

- The object is not marked as 'obsolete'

## Prerequisites
itop-structure/3.0.0 or newer

## Usage

## Notice
If running earlier versions of iTop 3.0.x, you might have to change the following line in action.class.inc.php:

```php
# Line 266, function FindRecipients
# Existing: 
protected function FindRecipients($sRecipAttCode, $aArgs)
{
    $sOQL = $this->Get($sRecipAttCode);
    if (strlen($sOQL) === 0) return '';
    // ......
}

# Change the following line
if (strlen($sOQL) == '') return '';

# To
if (strlen($sOQL) === 0) return '';
```

## Release

1. Gå till **CI/CD → Pipelines** i GitLab för det här repot.
2. Klicka **Run pipeline** på grenen `main`.
3. Valfritt: sätt variabeln `VERSION` (t.ex. `1.3.0`) om du vill välja version själv. Lämnas den tom höjs versionen automatiskt ett steg (t.ex. `1.2.0` → `1.3.0`).
4. Kör pipelinen och tryck igång det manuella jobbet **release**.

### Vad som händer sen (automatiskt)

- Ett nytt versionstagg skapas och paketet publiceras till registret.
- `hit-itop-docker` uppdaterar automatiskt sin `composer.json` med den nya versionen för just det här paketet - övriga paket påverkas inte.
- En ny Docker-image byggs, taggad med ett build-ID. Byggloggen visar vilka versioner som användes, och imagen innehåller en SBOM (`/opt/itop/sbom.json`) med samma build-ID.

### Driftsätta till TST

När imagen är byggd väntar jobbet **deploy-staging** i `hit-itop-docker` på manuellt godkännande innan den rullas ut till TST-klustret.

### Testköra en annan version (utan att releasa)

Du kan köra `hit-itop-docker`-pipelinen manuellt med variabeln `PACKAGE_OVERRIDES` för att bygga en engångs-image med andra versioner, t.ex. `hoglandetsit/hit-itop-organization:1.2.3`. Det sparas inte i `composer.json`.
