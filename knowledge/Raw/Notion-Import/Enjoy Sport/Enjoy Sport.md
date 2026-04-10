# Working Diary

#### Enjoy sport

|Name|Tags|
|---|---|
|[[07-11-2023]]||
|[[09-22-2023]]||
|[[10-11-2023]]||
|[[13-12-2023]]||
|[[14-12-2023]]||
|[[15-12-2023]]||
|[[18-12-2023]]||

  
  

# FM

https://www.figma.com/file/ZUBwIpNU9R7fS7WLgkjccG/Enjoy-Sport-Ticket?type=design&node-id=8-53&mode=design&t=bA9YLDxuqF2lSepA-0

  

# Jira

confluence: [https://belikesport.atlassian.net/wiki/spaces/enjoysport/overview?homepageId=327895](https://belikesport.atlassian.net/wiki/spaces/enjoysport/overview?homepageId=327895)

jr: [https://enjoysports.atlassian.net/jira/software/projects/ET/boards/1](https://enjoysports.atlassian.net/jira/software/projects/ET/boards/1)

# DB Design

  

acc: didimaster165@gmail.com

[https://dbdiagram.io/d/65464e0c7d8bbd64657ab3a0](https://dbdiagram.io/d/65464e0c7d8bbd64657ab3a0)

  

acc: vtdien96@gmail.com

[https://app.diagrams.net/#G1OTP0Mdi-CYP0lUIStpaaN-5c45BMTeFF](https://app.diagrams.net/#G1OTP0Mdi-CYP0lUIStpaaN-5c45BMTeFF)

[g](https://app.diagrams.net/#G1OTP0Mdi-CYP0lUIStpaaN-5c45BMTeFF)o

  

config: [https://code.enjoysport.vn/ticket/stg-cicd-manifest/-/blob/main/api-ticket/configmap.yaml?ref_type=heads](https://code.enjoysport.vn/ticket/stg-cicd-manifest/-/blob/main/api-ticket/configmap.yaml?ref_type=heads)

K8s: [https://argocd-stg.enjoysport.vn/applications](https://argocd-stg.enjoysport.vn/applications?showFavorites=false&proj=&sync=&autoSync=&health=&namespace=&cluster=&labels=)

  

# Insight

[https://worldsmarathons.com/](https://worldsmarathons.com/)

[[World marathon]]

[https://actiup.net/](https://actiup.net/)

[[Active Up]]

  

|   |   |   |   |   |   |
|---|---|---|---|---|---|
||Active Up|WM|IRace|Ticket box|Tóm lại|
||Cần Đăng nhập|Không cần|Cần Đăng nhập|Cần đăng nhập||
||Register With GG||Phải tự tạo, ko support OAuth( còn ko verify acc =)))|OAuth||
|Event|{  <br>"id": "65239f2faa2d9bbbb501bf0c",  <br>"event_slug": "dalat-ultra-trail-2024",  <br>"min_price": 650000,  <br>"square_url": "https:\/\/pix.raceez.com\/2023\/10\/13\/600x600.png",  <br>"banner_url": "https:\/\/pix.raceez.com\/2023\/10\/13\/1440x600.jpg",  <br>"selling_type": "selling",  <br>"categories": [  <br>{  <br>"id": "5dac1a27d274a435fe01e8c9",  <br>"cat_id": 2,  <br>"name_en": "Trail Running"  <br>}  <br>],  <br>"special_type": "hot",  <br>"is_ignore_participant_step": false,  <br>"event_type": "sports",  <br>"frequency_of_sell": "once",  <br>"start_date": "2024-03-15 08:00:00",  <br>"end_date": "2024-03-17 10:00:00",  <br>"name_en": "Dalat Ultra Trail 2024",  <br>"short_place_en": "The Valley of Love, Da Lat City",  <br>"short_description_en": "DALAT ULTRA TRAIL 2024 \u2013 JOURNEY OF A CONQUEROR Trail running is freedom of body and mind.\nIt\u2019s also for ALL, regardless of ability or background.\nTrail running is a way of life, a lifetime passion that is shared across the globe.\n\nDalat Ultra Trail is a RACE available to those who want to take a step into the unknown and immerse themselves in nature. Whatever your skill level or goals, giving it a shot is a good idea because of how fulfilling, easy, and inspiring it is."  <br>},||||Nếu phần register chứa product thì những cái còn lại gồm những gì? bố cục bao nhiêu phần?|
||- Active Up đang tách làm 2, tức là 1 trang giới thiệu event và 1 trang bán sản phẩm riêng|WM đang gọp cả 2 trang làm 1.|||Theo như design đang là sẽ chung 1 page. Giờ 2 option → gắn 1 ít và code 1 ít|
|||||||
|||||||
|Questionaire|Cách customize field tốt, 5km có áo, 21 ko|Fetch hết có vẻ như dùng cung|fetch toàn file js|Fetch bạo vì có danh sách chỗ ngồi|Mình có nên có type như AU để sau này dễ custome ( seat thì có chọn, ko thì thôi)  <br>Chỗ type nếu là option thì phải có thêm field option ds gồm có (value, label)  <br>"type": "virtual_event",  <br>"type": "text",  <br>"type": "select",  <br>"type": "date",|
|||||||
|||||||
|flow checkout||||||

  

# Task Decouple

#### Enjoy sport task

|Name|Assign|Date|Modules|Related Enjoy sport task|Status|T-Shirt|Type|
|---|---|---|---|---|---|---|---|
|[[Users Login]]||November 3, 2023 → November 6, 2023|users|[[User Profile]]|Not started||Task|
|[[Users Register]]||November 3, 2023 → November 6, 2023|users|[[User Profile]]|Not started||Task|
|[[Users Reset password]]||November 3, 2023 → November 6, 2023|users|[[User Profile]]|Not started||Task|
|[[Users Profile]]||November 3, 2023 → November 6, 2023|users|[[User Profile]]|Not started||Task|
|[[Events List events at home page]]||November 2, 2023 → November 5, 2023|events||Not started|L|Task|
|[[Detail events API]]|||events||Not started|M|Task|
|[[Create orders]]|||orders, products||Not started|M|Task|
|[[Update order]]|||orders, products|[[Pay orders]], [[Ticket checkout]]|Not started|M|Task|
|[[Checkout order]]|||orders, products||Not started|M|Task|
|[[Pay orders]]|||payments|[[Update order]], [[Ticket Payment]]|Not started|L|Task|
|[[Add on]]|||orders, payments|[[Product Management]]|Not started||Task|
|[[API fetch payment methods]]||||[[Ticket Payment]]|Not started||Task|
|[[Integrate with payment methods]]||||[[Ticket checkout]]|Not started||Task|
|[[Store payer + VAT]]||||[[Ticket Payment]]|Not started||Task|
|[[Orders management]]|||orders, payments, users|[[Order management]]|Not started||Task|
|[[change ticket product (cự ly)]]|||tickets|[[Ticket management]]|Not started||Task|
|[[Change ticket questionair]]|||tickets|[[Ticket management]]|Not started||Task|
|[[CRUD Events]]||||[[Admin Feature]]|Not started||Task|
|[[CRUD Users]]||||[[Admin Feature]]|Not started||Task|
|[[CRUD Products]]||||[[Admin Feature]], [[Product Management]]|Not started||Task|
|[[Send mail]]|||tickets|[[Ticket management]], [[Ticket checkout]]|Not started|S|Task|
|[[CRUD tickets]]|||||Not started||Task|
|[[integration payment solution]]|||orders, payments|[[Ticket checkout]]|Not started||Task|
|[[CRUD orders table]]|||orders||Not started||Task|
|[[CRUD payments table]]|||payments||Not started||Task|
|[[CRUD Marketing Campaigns]]|||||Not started|M|Task|
|[[Auto filling questionnaire]]|||||Not started||Task|
|[[Send mail confirm payment]]||||[[Ticket checkout]]||S|Task|