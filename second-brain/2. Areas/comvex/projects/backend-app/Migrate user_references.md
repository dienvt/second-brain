```
php artisan tinker --execute="
use Digima\Services\DatabaseManager;
use Digima\Models\Account\User;

DatabaseManager::connectToAccountDatabase(1); // change account id
User::withTrashed()->chunk(200, function(\$users){
	foreach (\$users as \$u) {
		try { \$u->remap(); } catch (\Throwable \$e) { \$u->map(); }
	}
});
"
```


```
php artisan tinker --execute='
use Digima\Services\DatabaseManager;
use Digima\Models\Account\User;

$accountId = 1;
$currentEmail = "admin1@example.com";   // user to find
$newEmail = "admin1@example.com";       // set new email here if changing
$newPassword = "new-Passw0rd";          // set desired password

if (!DatabaseManager::connectToAccountDatabase($accountId)) {
    throw new Exception("Cannot connect to account DB: {$accountId}");
}

$user = User::query()->where("email", $currentEmail)->first();

if (!$user) {
    throw new Exception("User not found: {$currentEmail} in account {$accountId}");
}

$user->email = $newEmail;
$user->password = $newPassword; // hashed by model mutator
$user->save();
$user->remap(); // sync core.user_references

echo "Updated user id={$user->id}, email={$user->email}\n";
'
```

