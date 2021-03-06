## Booking

The booking process allows you to book an event. Once you have found the _Event_ you want to book, using the search or browsing the catalogues, for a successful booking you have to follow this flow.

We assume you already retrieved the all the infomation for the _Event_ using the call:

```
GET /api/v3/events/{id}
```

### Booking flow

![Booking](http://musement.s3.amazonaws.com/documentation_images/booking.png)

1. Search for the available dates for the _Event_. | [Test it](https://thack.musement.com/documentation#get--api-v3-events-{id}-dates.{_format})

Returns a list of all days with availabitiy for a specific _Event_ over a period of time.

#### _Request_
```
GET /api/v3/events/{id}/dates
```

To specify the period you can use the parameters `from_date` and `end_date`. If none of this parameters are specified `from_date` is set to the current day. If `end_date` is not specified then `end_date` is set same as `from_date`.

**Examples:**

*** No parameters *** 

`from_date` and `end_date` are set as the current day. This call return the current date if there is availabitiy an empty array otherwise.

```
GET /api/v3/events/497/dates
```

*** With parameters *** 

Search for available dates for dicember 2016

```
GET /api/v3/events/497/dates?from_date=2016-12-01&end_date=2016-12-31
```

**Example Response:**

The response is a collection of date

```
[
  {
    "date": "2016-01-14"
  },
  {
    "date": "2016-01-25"
  }
]
```

### 3 Find tickets available for selected day

Once you found the date the next step is search for all available tickets for that day. Please note that for the same event different days can have different type of tickets.

#### _Request_
```GET /events/{id}/dates/{YYYY-MM-DD}``` | [Test it]

The response is a collection of items containing : 

 - `datetime` - The date and time. For a single day an event can have more starting time.
 - `seats` - A collection of all available `seats` (AKA `tickets`). Here the seat structure
 - `open_ticket` - If true the event is an `Open ticket`
 - `availability` - Number of place avaialble
 - `languages` - The event can be available in different language. Different time can have different languages

`seat` structure:

 - `id` - Unique seat identifier
 - `price_tag` - Give info about the person the ticket is for. Ticket type (`name`) and type of person (`group`) the `seat` is for. Please note that for the same date and time the combination of `name` and `group`  is unique but you can have more seat with the same `group` or `name`
 - `max_buy` and `min_buy` - Maximum and minimum number of ticket buyiable in a single transaction
 - `raw_price` and `retail_price` - Internal and public price

#### _Example_

```GET /api/v3/events/497/dates/2016-01-15```

Response

``` 
[
  {
    "datetime": "2016-01-14T13:45:00+0000",
    "seats": [
      {
        "id": 8216618,
        "price_tag": {
          "name": "Full price",
          "group": "Tour",
          "locale": "en"
        },
        "max_buy": 15,
        "min_buy": 1,
        "raw_price": {
          "currency": "EUR",
          "value": 40,
          "formatted_value": "€ 40.00"
        },
        "retail_price": {
          "currency": "EUR",
          "value": 40,
          "formatted_value": "€ 40.00"
        }
      }
    ],
    "open_ticket": false,
    "availability": 9,
    "languages": [
      "en"
    ]
  }
]
```

### Create the order

When the cart is completed (**with customer info filled in**) and you want to finalize it (proceed with payments and check the availability one last time) 

```
POST /api/v3/orders

{
    "cart_id": 123456
}
```

### Finalize the order

If the order amount is equal to 0 or you use your payment gateway, use the flow without payment (ask for activation). Otherwise follow the Payment flow in 4.4.2.


#### Without Payment Flow

Request ```POST /api/v3/payments/no-payment```

#### With Payment Flow

##### Braintree Payments

You have to integrate Braintree SDK

Note: in QA endpoint use credit card numbers contained in this link https://developers.braintreepayments.com/reference/general/testing/ruby#credit-card-numbers 

###### Step 1 
GET the token

Request ```POST /api/v3/payments/braintree/token```
Response ```{ "nonce": "eyJ2ZXJzaW9uIjoyLCJhdXRob3JpemF0aW9uRmluZ2VycHJpbnQiOiI3MDVjY2JlZThmZDRkNzI3MWQ2NTJjZDVjOTZjZDAyOTA5YWEzZDA4ZTJhN2ExNDVlNzc3MTI3MWYxMTUzMDIxfGNyZWF0ZWRfYXQ9MjAxNi0wMi0wM1QxMzowNzowOS43NjQ5NDQ2NjIrMDAwMFx1MDAyNm1lcmNoYW50X2lkPXE5MjdzdGt5cjRtZHc1MnhcdTAwMjZwdWJsaWNfa2V5PWNzMjJkMjJzbjc2ano4YzYiLCJjb25maWdVcmwiOiJodHRwczovL2FwaS5icmFpbnRyZWVnYXRld2F5LmNvbTo0NDMvbWVyY2hhbnRzL3E5MjdzdGt5cjRtZHc1MngvY2xpZW50X2FwaS92MS9jb25maWd1cmF0aW9uIiwiY2hhbGxlbmdlcyI6WyJjdnYiXSwiZW52aXJvbm1lbnQiOiJwcm9kdWN0aW9uIiwiY2xpZW50QXBpVXJsIjoiaHR0cHM6Ly9hcGkuYnJhaW50cmVlZ2F0ZXdheS5jb206NDQzL21lcmNoYW50cy9xOTI3c3RreXI0bWR3NTJ4L2NsaWVudF9hcGkiLCJhc3NldHNVcmwiOiJodHRwczovL2Fzc2V0cy5icmFpbnRyZWVnYXRld2F5LmNvbSIsImF1dGhVcmwiOiJodHRwczovL2F1dGgudmVubW8uY29tIiwiYW5hbHl0aWNzIjp7InVybCI6Imh0dHBzOi8vY2xpZW50LWFuYWx5dGljcy5icmFpbnRyZWVnYXRld2F5LmNvbSJ9LCJ0aHJlZURTZWN1cmVFbmFibGVkIjpmYWxzZSwicGF5cGFsRW5hYmxlZCI6dHJ1ZSwicGF5cGFsIjp7ImRpc3BsYXlOYW1lIjoiTXVzZW1lbnQiLCJjbGllbnRJZCI6IkFkcTdxSTJrS2NNTEFIQjRGRDRac1JxNC13VGp0eUxnckhRdllSTGMtQ1k0MGFmZlh2dUYwUHpLRUJSSVZfRktJeTVaNmJvREZpY1A2ZmIzIiwicHJpdmFjeVVybCI6Imh0dHBzOi8vd3d3Lm11c2VtZW50LmNvbS9lbi9wL3ByaXZhY3kiLCJ1c2VyQWdyZWVtZW50VXJsIjoiaHR0cHM6Ly93d3cubXVzZW1lbnQuY29tL2VuL3AvdGVybXMtY29uZGl0aW9ucyIsImJhc2VVcmwiOiJodHRwczovL2Fzc2V0cy5icmFpbnRyZWVnYXRld2F5LmNvbSIsImFzc2V0c1VybCI6Imh0dHBzOi8vY2hlY2tvdXQucGF5cGFsLmNvbSIsImRpcmVjdEJhc2VVcmwiOm51bGwsImFsbG93SHR0cCI6ZmFsc2UsImVudmlyb25tZW50Tm9OZXR3b3JrIjpmYWxzZSwiZW52aXJvbm1lbnQiOiJsaXZlIiwidW52ZXR0ZWRNZXJjaGFudCI6ZmFsc2UsImJyYWludHJlZUNsaWVudElkIjoiQVJLcllSRGgzQUdYRHpXN3NPXzNiU2txLVUxQzdIR191V05DLXo1N0xqWVNETlVPU2FPdElhOXE2VnBXIiwiYmlsbGluZ0FncmVlbWVudHNFbmFibGVkIjpmYWxzZSwibWVyY2hhbnRBY2NvdW50SWQiOiJtdXNlbWVudEVVUiIsImN1cnJlbmN5SXNvQ29kZSI6IkVVUiJ9LCJjb2luYmFzZUVuYWJsZWQiOmZhbHNlLCJtZXJjaGFudElkIjoicTkyN3N0a3lyNG1kdzUyeCIsInZlbm1vIjoib2ZmIiwiYXBwbGVQYXkiOnsic3RhdHVzIjoicHJvZHVjdGlvbiIsImNvdW50cnlDb2RlIjoiVVMiLCJjdXJyZW5jeUNvZGUiOiJFVVIiLCJtZXJjaGFudElkZW50aWZpZXIiOiJtZXJjaGFudC5jb20ubXVzZW1lbnQubXVzZW1lbnRwcm9kIiwic3VwcG9ydGVkTmV0d29ya3MiOlsidmlzYSIsIm1hc3RlcmNhcmQiXX18" }```

###### Step 2 

Using Braintree SDK, ask the user for payment information (credit card, paypal etc). For more details please visit: https://developers.braintreepayments.com/

###### Step 3 

After the user fills out the braintree payment form, ask us to process the payment and close the order 

Request ```POST /api/v3/payments/braintree/payment```

Request Body 
```
{
  "nonce" : "nonce_text",
  "orderId" : 123456
}
```

### 3.7 Retrieve the voucher

If you used the login 2.1.2 you can retrieve the voucher with the Request

```GET /customer/me/orders```

Otherwise we will send it to your email address.
