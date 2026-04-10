---
title: "Authorization"
date: 2026-01-14
tags:
  - projects
---

Authorization

Authorize now, capture later

  

# Authorization

Here's a step-by-step breakdown of the process:

1. **Initiation**: When a cardholder presents a credit card for a purchase, either by swiping, inserting (for EMV chips), or entering the card information online, the merchant's point-of-sale system or online shopping cart initiates an authorization request.
2. **Transmission**: The merchant's system sends the transaction details, including the card number, expiration date, CVV/CVC code, and transaction amount, to the payment gateway or processor.
3. **Processor to Network**: The payment processor forwards the authorization request to the card network (e.g., Visa, MasterCard, American Express).
4. **Network to Issuing Bank**: The card network then sends the request to the card's issuing bank.
5. **Approval or Decline**: The issuing bank checks the available credit or funds in the account. It also checks for any potential security issues, such as whether the card has been reported lost or stolen. Based on these checks, the bank will either approve or decline the transaction. An approved transaction will reserve the specified amount on the cardholder's account, ensuring the funds are set aside for when the merchant later settles the transaction.
6. **Response**: The decision (either approval or decline) is then sent back through the card network, to the payment processor, and finally to the merchant's point-of-sale system or online shopping cart.
7. **Communication to Customer**: The merchant's system will then display or communicate the transaction's status to the customer. If approved, the transaction goes through, but the merchant hasn't yet collected the funds. If declined, the purchase is not completed, and the customer is typically notified of the reason for the decline when possible.
8. **Settlement**: At the end of the day or another predefined period, the merchant sends all approved transactions in a batch to their payment processor for settlement. This is when the actual transfer of funds from the cardholder's bank to the merchant's bank takes place.

Note: A pre-authorization can also be used in situations where the final amount may not yet be known, such as when checking into a hotel or renting a car. In these cases, a certain amount is "held" on the card until the final transaction amount is determined, and then the transaction is settled for the actual amount.

It's essential for merchants to understand the authorization process to manage their payment systems effectively and minimize transaction disputes and chargebacks.

  

# Authorize now, capture later

1. **Authorization**: This is the initial step where the acquiring bank (or the merchant's bank) sends a request to the issuing bank (or the cardholder's bank) to check if the cardholder's account has sufficient funds or credit limit for the transaction. If approved, the specified transaction amount is reserved, but no actual funds are moved. This "hold" reduces the cardholder's available balance, ensuring that they don't spend the reserved amount elsewhere before the transaction is settled.
2. **Delayed Settlement (Capture)**: After an authorization, the merchant doesn't immediately capture or settle the transaction. The actual movement of funds from the cardholder's bank to the merchant's bank account occurs during the settlement process. Depending on the type of transaction and the merchant's preference, this settlement can happen almost immediately after authorization or be delayed.
    - For instance, in e-commerce scenarios where physical goods are involved, a merchant might authorize the card when an order is placed but only capture (settle) the transaction once the item is shipped.
    - In industries like hotels or car rentals, authorization might occur when the service is first engaged (e.g., checking into a hotel), but settlement might not occur until the service is completed (e.g., checking out of the hotel).
3. **Voiding or Releasing the Authorization**: If, for some reason, the merchant decides not to go through with the transaction, they can void the authorization. This action informs the issuing bank that the merchant won't be capturing the transaction, and the reserved funds should be released. Depending on the bank's policies, the release of this "hold" on the cardholder's funds might be immediate or take a few days.
4. **Expiration of Authorization**: Authorizations don't last indefinitely. If a merchant doesn't capture the transaction within a specified period (often several days, but this can vary depending on the merchant agreement and the card network policies), the authorization will automatically expire, and the hold on the funds will be released.

To implement this "authorize now, capture later" approach, the merchant's payment processing system must support both separate authorization and capture steps. Most modern payment gateways and processors provide this functionality. It's crucial for merchants using this method to understand their responsibilities, particularly the need to capture the transaction within the authorization window and the importance of voiding any unnecessary authorizations to release holds on their customers' funds.