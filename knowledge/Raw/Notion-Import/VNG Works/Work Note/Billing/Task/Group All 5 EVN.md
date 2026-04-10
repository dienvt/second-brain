---
Assign: DDien Vo
Status: Completed
---
```SQL
USE `zpCPSPlatformAdmin`;
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


INSERT INTO `SupplierGroup` (`supplierGroupID`, `supplierGroupName`, `status`, `icon`) VALUES ('3', 'Tập đoàn điện lực Việt Nam', '1', 'EVN');

UPDATE `Supplier` SET `supplierGroupID` = '3' WHERE (`supplierID` = '1');
UPDATE `Supplier` SET `supplierGroupID` = '3' WHERE (`supplierID` = '3');
UPDATE `Supplier` SET `supplierGroupID` = '3' WHERE (`supplierID` = '2');
UPDATE `Supplier` SET `supplierGroupID` = '3' WHERE (`supplierID` = '4');
UPDATE `Supplier` SET `supplierGroupID` = '3' WHERE (`supplierID` = '5');
```