  

  

## Field

Using `columninfo` in `embedData` field in `OrderEntity` and `zlppaymentid` field

## EmbedData

```Bash
type EmbedData struct {
	ColumnInfo    string `json:"columninfo"`
	Promotioninfo string `json:"promotioninfo"`
	Zlppaymentid  string `json:"zlppaymentid"`
}
```

## ColumnInfo

### Standard format

- Provider
    
    `_SCTV_`_,_`_TAYNINH_WATER_`_,_ `_CANTHO_WATER_`
    

```Bash
type ColumnInfo struct {
	Customercode string `json:"customercode"`
	Customername string `json:"customername"`
	Address      string `json:"address"`
	Data         string `json:"data"` // JSON of []ColumnInfoData
	// anything else
}

type ColumnInfoData struct {
	Month  string `json:"month"`
	Amount string `json:"amount"`
}
```

### Specific format

- Provider
    
    `ELEC`,
    
      
    

```Bash
type ColumnInfo struct {
	Customercode string `json:"customercode"`
	Customername string `json:"customername"`
	Address      string `json:"address"`
	Data         string `json:"data"` // JSON of []ColumnInfoData
	// anything else
}

type ColumnInfoData struct {
	Month  string `json:"month"`
	Amount string `json:"amount"`
}
```

  

## Sample data

```Bash
"embedData": "{\"columninfo\":\"{\\\"address\\\":\\\"***0174\\\",\\\"companyname\\\":\\\"FC\\\",\\\"customercode\\\":\\\"20210218-5986976\\\",\\\"customername\\\":\\\"PHẠM THỊ LIỂU\\\",\\\"data\\\":\\\"[{\\\\\\\"duedate\\\\\\\":\\\\\\\"20/09/2021\\\\\\\",\\\\\\\"month\\\\\\\":\\\\\\\"09/2021\\\\\\\",\\\\\\\"amount\\\\\\\":49625,\\\\\\\"minamount\\\\\\\":1000}]\\\"}\",\"promotioninfo\":\"{\\\"productinfo\\\":[{\\\"category\\\":\\\"701\\\",\\\"count\\\":0,\\\"manufactory\\\":\\\"FECREDIT\\\",\\\"sku\\\":\\\"20210218-5986976\\\",\\\"subcategory\\\":\\\"20210218-5986976\\\"}]}\",\"zlppaymentid\":\"FCLOAN\"}"
```

  

# SSC

ColumnInfo  
{  
"customercode": "1202043422000226",  
"customername": "Luu Nguyen Huy Hoang",  
"address": "capnhatsau",  
"data": "{\"fee\":1000,\"bills\":[{\"month\":\"07/2022\",\"amount\":10000,\"fee\":1}]}"  
}  
  
data JSON:  
{  
"fee": 1000,  
"bills":  
[  
{  
"month": "07/2022",  
"amount": 10000,  
"fee": 1  
}  
]  
}