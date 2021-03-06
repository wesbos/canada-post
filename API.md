# API

This class loosely follows the API available from CanadaPost. See [their documentation](https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/default.jsf) for more information.

## Property Naming & Result Objects
The Canada API returns XML elements that are dash-separated.
These are converted to lowerCamelCase Javascript objects, and vice-versa when calling a method.

This means that instead of providing a property like `show-packing-instructions`, you may provide `showPackingInstructions`,
and it will be converted before being sent with the request.

Result objects are somewhat normalized, to reduce the amount of unncessary XML information/structure that is generated.

## Promises
_All_ Candada Post API methods return a promise. Old-style callbacks are not supported.

## API
- [CanadaPostClient](#CanadaPostClient)
    - [`new CanadaPostClient(userId, password, [customer, [lang]])`](#new-canadapostclientuserid-password-customer-lang---canadapostclient)
    - [`.discoverServices(originPostalCode, destinationCountry, [destinationPostalCode])` -> `Promise`](#discoverservicesoriginpostalcode-destinationcountry-destinationpostalcode---promise)
    - [`.getRates(scenario)` -> `Promise`](#getratesscenario---promise)
    - [`.createNonContractShipment(shipment)` -> `Promise`](#createNonContractShipmentshipment---promise)
    - [`.refundNonContractShipment(id, email) -> `Promise`](#refundNonContractShipmentid-email---promise))
    - *(additional documentation coming soon)*

## CanadaPostClient
The main class for working with the Canada Post API

##### `new CanadaPostClient(userId, password, [customer, [lang]])` -> `CanadaPostClient`
Creates a new instance of the CanadaPostClient class, that will authenticate requests with the provided username and password.

Arguments:

- `userId` (String) - Your [username](https://www.canadapost.ca/cpotools/apps/drc/registered?execution=e3s1)
- `password` (String) - Your [password](https://www.canadapost.ca/cpotools/apps/drc/registered?execution=e3s1)
- `customer` (String) [optional] - Your customer ID. This will be used by default for requests, if one is required in the path.
- `lang` (String) [optional] - The language of the responses. Should be one of en-CA, or fr-CA.

Example:
```javascript
const cpClient = new CanadaPostClient('b4c32...', 'c0009a...');

let result = cpClient.discoverServices('P0ST4L','US');
```

Returns: `CanadaPostClient instance`

***

##### `.discoverServices(originPostalCode, destinationCountry, [destinationPostalCode])` -> `Promise`
Discovers available services for creating a shipment between the origin postal code, and destination.

See: https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/rating/getrates/discoverservices.jsf

Arguments:

- `originPostalCode` (String) - The origin postal code, within Canada, that the item is being shipped from
- `destinationCountry` (String) - The ISO2 country code the package is being shipped to
- `destinationPostalCode` (String) [optional] - If the destination is within Canada, provide the destination postal code for more accurate results.

Returns: `Promise`
Resolves: `Array` - An array of services codes for shipping from the origin to the destination

***

##### `.getRates(scenario)` -> `Promise`
Gets the rates for a given mailing scenario.

See: https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/rating/getrates/default.jsf

Arguments:

- `scenario` (Object) - An object that represents the mailing scenario. See [this page](https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/rating/getrates/default.jsf) for info. Common fields include the following:
  - `parcelCharacteristics` (Object) - An object that describes the parcel
    - `weight` (Number) - The weight of the parcel in kilograms
    - `dimensions` (Object) - The dimensions of the parcel.
      - `length` (Number) - Longest dimension in centimeters, to one decimal point
      - `width` (Number) - Second longest dimension in centimeters, to one decimal point
      - `height` (Number) - Shortest dimension in centimeters, to one decimal point
  - `originPostalCode` (String) - The origin postal code, within Canada, that the item is being shipped from
  - `destination` (Object) - The destination address, with **only one** of the following child attributes
    - `domestic` (Object) - For Canadian shipments
      - `postalCode` (String) - The destination postal code
    - `unitedStates` (Object) - For US shipments
      - `zipCode` (String) - The destination ZIP code
    - `international` (Object) - For other, worldwide destinations
      - `countryCode` (String) - The ISO 3166-1 alpha-2 country code (e.g. GB, MX). See the [International Shipping Chart](https://www.canadapost.ca/tools/pg/prices/RCRZ-e-ISC.pdf)

Returns: `Promise`
Resolves: `Array` - An array of available services and their prices for shipping from the origin postal code to the destination

***

##### `.createNonContractShipment(shipment)` -> `Promise`
Creates a non-contract shipment

See: https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/onestepshipping/createshipment.jsf

Arguments:
- `shipment` (Object) - An object that represents the shipment. See [this page](https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/onestepshipping/createshipment.jsf) for info. Common fields include the following:
  - `requestedShippingPoint` (String) - The origin postal code from where the parcel will be shipped.
  - `deliverySpec` (Object) - The specifications of the delivery
    - `serviceCode` (String) - The code to use (e.g. DOM.EP, USA.XP)
    - `sender` (Object) - The sender's address information
      - `name` (String)
      - `company` (String)
      - `contactPhone` (String)
      - `addressDetails` (Object)
        - `addressLine1` (String)
        - `addressLine2` (String)
        - `city` (String)
        - `provState` (String)
        - `postalZipCode` (String)
    - `destination` (Object) - The recipient's address information
      - `name` (String)
      - `company` (String)
      - `clientVoiceNumber` (String) - Recipient's phone number (required for international shipments)
      - `addressDetails` (Object)
        - `addressLine1` (String)
        - `addressLine2` (String)
        - `city` (String)
        - `provState` (String)
        - `postalZipCode` (String)
        - `countryCode` (String)
    - `parcelCharacteristics` (Object) - An object that describes the parcel
      - `weight` (Number) - The weight of the parcel in kilograms
      - `dimensions` (Object) - The dimensions of the parcel.
        - `length` (Number) - Longest dimension in centimeters, to one decimal point
        - `width` (Number) - Second longest dimension in centimeters, to one decimal point
        - `height` (Number) - Shortest dimension in centimeters, to one decimal point
    - `preferences` (Object) - Your shipment preferences
      - `showPackingInstructions` (String) - true/false for whether to show packing instructions on the generated shipping label

Returns: `Promise`
Resolves: `Object` - An object representing the created shipment, with the shipmentId, trackingPin, labels, and other URLs.

***

##### `.refundNonContractShipment(id, email)` -> `Promise`
Refunds a non-contract shipment

See: https://www.canadapost.ca/cpo/mc/business/productsservices/developers/services/onestepshipping/shipmentrefund.jsf

Arguments:
- `id` (String) - The shipment ID to refund
- `email` (String) - The account holder's e-mail address

Returns: `Promise`
Resolves: `Object` - An object with a serviceTicketDate, and serviceTicketId to indicate your refund request has been recieved. You can use the serviceTicketId when communicating with Canada Post

Note that refunds take a few days to process, and a successful response here does not indicate the refund has completed.