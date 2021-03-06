# Rebilly JS Token Library

Rebilly.js powers your checkout form and removes the need to send sensitive customer information directly to your servers. Use the library to generate payment tokens to reduce the scope of PCI DSS compliance.

[![npm](https://img.shields.io/npm/v/rebilly-js-token.svg)](https://www.npmjs.com/package/rebilly-js-token)
[![Build Status](https://travis-ci.org/Rebilly/rebilly-js-token.svg?branch=master)](https://travis-ci.org/Rebilly/rebilly-js-token)
[![devDependencies Status](https://david-dm.org/Rebilly/rebilly-js-token/dev-status.svg)](https://david-dm.org/Rebilly/rebilly-js-token?type=dev)


### Rebilly API Spec
The library uses the payment token endpoint from the Rebilly API. See the [Rebilly API spec](https://rebilly.github.io/RebillyAPI/) for more details.

## Documentation
Visit the [GitHub pages](https://rebilly.github.io/rebilly-js-token/) for detailed documentation.

## Including Rebilly.js

Add Rebilly.js to your page using one of the following CDN providers, preferably at the bottom before the `</body>`. 

> Always use `HTTPS` when including the library.

#### UNPKG CDN

```html
<script src="https://unpkg.com/rebilly-js-token/dist/rebilly.js"></script>
```

#### jsDelivr CDN

```html
<script src="https://cdn.jsdelivr.net/npm/rebilly-js-token/dist/rebilly.min.js"></script>
```

## Usage

After including the library into your page, you must authenticate your API requests then define the data to use for the token and provide callback function.

### Authentication
Once included in your checkout page, authenticate your token requests using a signature generated by one of Rebilly's server-side SDKs using your secret API key. The authentication must happen before a token is to be created.

- [Rebilly PHP SDK](https://github.com/Rebilly/rebilly-php)
- [Rebilly .NET SDK](https://github.com/Rebilly/Rebilly-NET-SDK)
- [Rebilly JS SDK (Node)](https://github.com/Rebilly/rebilly-js-sdk)

```js
Rebilly.setAuth('authentication-signature-value');
```
 
### Creating a token
To create a token you must provide two parameters: the form or object literal with the payment instrument data (payment card or bank account) and a callback function that will receive the resulting token from the Rebilly API.

> Tip: when creating a token, prevent the default submission of the form until a value is returned by the API and injected into your page.

```js
Rebilly.createToken(Node|Object, Function)
```

#### Building the payment instrument data
The first parameter will be the payment instrument data. You can use either a form node in your page or a plain object literal.

##### Parse a form for the payment instrument
The library can look for field with the `data-rebilly` attribute and compile the data from your form directly. Specify the field name associated in Rebilly as `data-rebilly="fieldName"`.

You can omit providing a `method` field, the library will detect it based on which fields you specified.

> **PCI Compliance Note**: never define `name` attributes for the payment card fields in your form. This will prevent field data from showing up in your server logs.

```html
<form>
    <input data-rebilly="pan">
    <input type="number" data-rebilly="expYear">
    <input type="number" data-rebilly="expMonth">
    <input type="number" data-rebilly="cvv">
</form>
```

Using the form above the library will detect a payment card.

```js
var form = document.getElementsByTagName('form')[0];
Rebilly.createToken(form, callback);
```

##### Use an object literal
```js
var payload = {
    method: 'payment-card',
    paymentInstrument: {
        pan: '4111111111111111',
        expYear: '2022',
        expMonth: '12',
        cvv: '123'
    }
};
Rebilly.createToken(payload, callback);
```

##### Define the callback
The callback function should be used to inject the token returned by the API into your form. Once submitted, use the value in conjunction with one of the server-side SDKs to create the customer.

```js
// the token is returned as response.data.id
var callback = function (response) {
    // create a hidden input field
    var tokenField = document.createElement('input');
    tokenField.setAttribute('type', 'hidden');
    tokenField.setAttribute('name', 'payment-token');
    tokenField.value = response.data.id;
    // append to the form and submit to the server
    form.appendChild(tokenField);
    form.submit();
};

Rebilly.createToken(form, callback);
```

##### Callback Argument 
The argument received by the callback contains additional information on the API request and can be used to detect validation errors.

| Property | Type | Description |
| -------- | ---- | ----------- |
| error | boolean | Defines whether there was an error with the request or not. |
| message | string | The response message. Returns `success` if there was no errors, or the error message. |
| status | number | The status code returned by the response. |
| data | Object | The response data as returned by the API. The token is exposed as `data.id`. |
| xhr | Object | The raw XHR request object. |

## Development Commands

Build development `dist` folder without sourcemap
```
npm run build:dev
```
Build release `dist` folder with sourcemap (release)
```
npm run build:prod
```
Run all unit tests
```
npm run test
```
Watch unit tests and re-run on change
```
npm run test:watch
```
