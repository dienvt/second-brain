---
Assign: DDien Vo
Status: Completed
---
## STG

### Check list

## 3.5.0 - 2021-06-15  
### Added  

- [x] Group 5 EVN, detail script
- [ ] Get and deliver order -16, -17
- [ ] Maintain provider (done disable provider to customer)
- [x] Security bug
- [ ] GameCard and Postpaid app info  
      
    

  

  

```SQL
USE zpCPSPlatform_Admin;
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


INSERT INTO `ProviderApiConfig` (`section`, `key`, `value`) VALUES ('BTWater', 'status', '1'),
('CardStore', 'status', '1'),
('EVNGateway', 'status', '1'),
('EVNGatewayV2', 'status', '1'),
('EVNGatewayV3', 'status', '1'),
('FECredit', 'status', '1'),
('GLWater', 'status', '1'),
('HaNoiWater', 'status', '1'),
('HomeCredit', 'status', '1'),
('Htvc', 'status', '1'),
('HueWater', 'status', '1'),
('KPLUSGateway', 'status', '1'),
('Mirae', 'status', '1'),
('NamDinhWater', 'status', '1'),
('NhaBeProvider', 'status', '1'),
('Payoo', 'status', '1'),
('SSC', 'status', '1'),
('Savista', 'status', '1'),
('Shinhan', 'status', '1'),
('TDWATER', 'status', '1'),
('VINHLONGWATER', 'status', '1'),
('VNPAY', 'status', '1'),
('MCredit', 'status', '1'),
('PhuHoaTan', 'status', '1'),
('VNPT', 'status', '1'),
('VTVC', 'status', '1');
```

## PRO

### Check list

## 3.5.0 - 2021-06-15  
### Added  

- [x] Group 5 EVN, detail script
- [ ] Get and deliver order -16, -17
- [ ] Maintain provider (done disable provider to customer)
- [x] Security bug
- [ ] GameCard and Postpaid app info

  

  

```SQL
USE `zpCPSPlatformAdmin`;

INSERT INTO `ApiConfig` (`section`, `key`, `value`) VALUES ('UmMerchant', 'verifyInternalMerAccessTokenPath', 'verifyinternalmerchantaccesstoken');

INSERT INTO `ProviderApiConfig` (`section`, `key`, `value`) VALUES ('BTWater', 'status', '1'),
('CardStore', 'status', '1'),
('EVNGateway', 'status', '1'),
('EVNGatewayV2', 'status', '1'),
('EVNGatewayV3', 'status', '1'),
('FECredit', 'status', '1'),
('GLWater', 'status', '1'),
('HaNoiWater', 'status', '1'),
('HomeCredit', 'status', '1'),
('Htvc', 'status', '1'),
('HueWater', 'status', '1'),
('KPLUSGateway', 'status', '1'),
('Mirae', 'status', '1'),
('NamDinhWater', 'status', '1'),
('NhaBeProvider', 'status', '1'),
('Payoo', 'status', '1'),
('SSC', 'status', '1'),
('Savista', 'status', '1'),
('Shinhan', 'status', '1'),
('TDWATER', 'status', '1'),
('VINHLONGWATER', 'status', '1'),
('VNPAY', 'status', '1'),
('VNPT', 'status', '1'),
('VTVC', 'status', '1');


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

DROP TABLE `SupplierPattern`;
CREATE TABLE `SupplierPattern` (
 `appID` INT NOT NULL,
 `pattern` VARCHAR(45) NOT NULL,
 `supplierID` INT NOT NULL,
 `status` INT NOT NULL,
 `description` VARCHAR(255) NULL,
 `updatedBy` VARCHAR(45) NOT NULL,
 `updatedAt` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP() ON UPDATE CURRENT_TIMESTAMP(),
PRIMARY KEY (`appID`, `supplierID`));
INSERT INTO `SupplierPattern` (`appID`, `pattern`, `supplierID`, `status`, `updatedBy`) VALUES ('17', '^PC.*,^PP.*,^PQ.*', '1', '1', 'dienvt');
INSERT INTO `SupplierPattern` (`appID`, `pattern`, `supplierID`, `status`, `updatedBy`) VALUES ('17', '^PD.*', '2', '1', 'dienvt');
INSERT INTO `SupplierPattern` (`appID`, `pattern`, `supplierID`, `status`, `updatedBy`) VALUES ('17', '^PA.*,^PH.*,^PM.*,^PN.*', '3', '1', 'dienvt');
INSERT INTO `SupplierPattern` (`appID`, `pattern`, `supplierID`, `status`, `updatedBy`) VALUES ('17', '^PE.*', '4', '1', 'dienvt');
INSERT INTO `SupplierPattern` (`appID`, `pattern`, `supplierID`, `status`, `updatedBy`) VALUES ('17', '^PB.*,^PK.*', '5', '1', 'dienvt');
```