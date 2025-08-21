Nó achieve tiếng việt tiếng anh bằng field _en, _vi

  

1/Chọn vé > 2/Thông tin người mua > 3/Thông tin người đăng kí > 4/Thanh toán

1. Chọn vé: chọn số lượng vé thôi
2. Thông tin người mua: dinh tới orders xuất hoá đơn hay không? tên email người mua
    1. Tạo order và fetch về bằng API: [https://actiup.net/api/checkout/cart/mine/events/65239f2faa2d9bbbb501bf0c?includes=child_items,items,attendant_templates](https://actiup.net/api/checkout/cart/mine/events/65239f2faa2d9bbbb501bf0c?includes=child_items,items,attendant_templates)
3. Thông tin người đăng kí (đi với cái bib) hay nó gọi là checkout.
    
    1. Lúc checkout thì sẽ có field child_Items. Mỗi item trong đó có field attendant_template_id để FE biết cần lấy thôgn tin gì. có cả field "qty": 2, attendant (tham dự)
    2. product, is_sold_out, is_active
    3. Dùng field attendant_templates để định nghĩa các field
    4. Đa số sẽ phải định nghĩa ntn:
    
    {  
    "label_en": "Email",  
    "label_vi": "\u0110\u1ecba ch\u1ec9 email",  
    "number": "4",  
    "validation": "email",  
    "key": "email",  
    "type": "text",  
    "size": 6,  
    "require": true,  
    "is_lock_update_data": false,  
    "sub_label_en": "",  
    "sub_label_vi": "",  
    "sub_label_url": "",  
    "option": null  
    }, hoặc  
    
    {  
    "label_en": "ID\/Passport Number",  
    "label_vi": "CMND ho\u1eb7c h\u1ed9 chi\u1ebfu",  
    "number": "5",  
    "  
    **validation**": "length",  
    "key": "id_card",  
    "type": "text",  
    "size": 6,  
    "require": true,  
    "is_lock_update_data": false,  
    "sub_label_en": "",  
    "sub_label_vi": "",  
    "sub_label_url": "",  
    "option": {  
    "max": 20  
    }  
    },  
    
    field type:
    
    "type": "virtual_event",  
    "type": "text",  
    "type": "select",  
    "type": "date",  
    
      
    
      
    
    1. Lúc checkout có call API lấy method thanh toán: https://actiup.net/api/checkout/cart/mine/events/65239f2faa2d9bbbb501bf0c/payment/methods
    2. Thanh toán là xong thôi