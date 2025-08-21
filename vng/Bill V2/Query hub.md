- metrics data push cho ZDS 
- Monitor metrics and participant


# Sequence

```plantuml
title "Query"
autonumber

participant OrtherSystem as oths
queue AutoQuery as aqq

box "Internal Service" #LightGreen
participant Consumer
participant BusinessDomain
participant APIHanlder
end box

participant Provider
queue AutoQueryResponse as aqrq

=== pre ===
aqq -> Consumer: supplier_id, product_line, customer_code
Consumer -> BusinessDomain: call

oths -> APIHanlder: supplier_id, product_line, customer_code
APIHanlder -> BusinessDomain: call

=== in process ===

BusinessDomain -> BusinessDomain: check pre-request

BusinessDomain -> BusinessDomain: get provider

BusinessDomain -> Provider: do call inquiry

BusinessDomain -> BusinessDomain: observe response

BusinessDomain -> aqrq: publish response


```