---
Assign: DDien Vo
Status: Completed
---
# changelog

# Added

- maintain provider buy time

  

  

# Scrip

```SQL
USE zpCPSPlatformAdmin;

DROP TABLE IF EXISTS `ProviderDailyMaintain`;
CREATE TABLE `ProviderDailyMaintain` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `providerCode` VARCHAR(45) NOT NULL,
  `startHour` INT(2) NULL,
  `startMinute` INT(2) NULL,
  `endHour` INT(2) NULL,
  `endMinute` INT(2) NULL,
  `status` INT(1) NULL DEFAULT 0,
  `CreateBy` VARCHAR(45) NULL,
  `UpdateBy` VARCHAR(45) NULL,
  `CreateAt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `UpdateAt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`));



DROP TABLE IF EXISTS `ProviderDateMaintain`;
CREATE TABLE `ProviderDateMaintain` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `providerCode` VARCHAR(45) NOT NULL,
  `startDate` BIGINT(21) NULL,
  `endDate` BIGINT(21) NULL,
  `status` INT(1) NULL DEFAULT 0,
  `CreateBy` VARCHAR(45) NULL,
  `UpdateBy` VARCHAR(45) NULL,
  `CreateAt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `UpdateAt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`));
```