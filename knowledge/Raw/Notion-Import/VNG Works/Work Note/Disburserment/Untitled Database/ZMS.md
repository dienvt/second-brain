```SQL
ALTER TABLE `disbursement`.`orders` 
ADD COLUMN `partner_name` VARCHAR(256) NOT NULL DEFAULT '' AFTER `partner_wallet_id`,
ADD COLUMN `user_zalo_id` VARCHAR(32) NOT NULL DEFAULT '' AFTER `user_id`;
```

change file config

zms{

"enable": true,

}