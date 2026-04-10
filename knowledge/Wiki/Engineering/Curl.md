---
title: "Curl"
date: 2022-11-29
tags:
  - engineering
  - curl
---


curl POST 'https://backend.epass-vdtc.com.vn/crm2/api/v1/login' \
--insecure --location --request \
--proxy http://10.50.32.3:3128 \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=zalopay' \
--data-urlencode 'client_secret=zhnN33EubYp25JIky2ipHgh7ukhEhPj9' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'password=epaSS2202ZalOPay!@#' \
--data-urlencode 'username=zalopay'
