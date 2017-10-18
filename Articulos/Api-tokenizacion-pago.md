## API Tokenización y Pago

La integración mediante nuestra API te da un mayor control sobre el checkout en tu sitio, ya que el proceso se divide en dos etapas: Obtener un token de la tarjeta de crédito y realizar el cargo, y estas pueden ocurrir en momentos distintos.

 Los pasos para realizar la integración son:

- [1. Obtener un Token de Acceso](#1-obtener-un-token-de-acceso)
- [2. Crear una Intención de Captura](#2-crear-una-intención-de-captura)
- [3. Mostrar formulario de Captura de tarjeta](#3-mostrar-formulario-de-captura-de-tarjeta)
- [4. Intención de Pago](#4-intención-de-pago)
- [5. Realizar el cargo a la tarjeta](#5-cómo-realizar-el-cargo-a-la-tarjeta)

#### 1. Obtener un Token de Acceso

Al completar el registro de la aplicación (Alta del Comercio), obtendrás dos llaves con las cuales te podrás autenticar en el sistema, a estas les llamamos **client_id** (identificador) y **client_secret** (Clave Secreta).

![Ejemplo de client_id y client_secret](Portal00.jpg)

Con estas credenciales podrás obtener el **token de acceso** llamando a la **API de Autenticación** de la siguiente forma:

```export CLIENT_ID=772778451507664600860
export CLIENT_ID=772778451507664600860
export CLIENT_SECRET=3f9IecqBbiZew3sOQhzSPF62xHdgGzNHx2kShWpaqzaT
curl -v -X POST https://quickpay-connect-sso.azurewebsites.net/oauth2/v2/token \
 -H "Content-Type:application/x-www-form-urlencoded" \
 -H "Authorization: Basic $CLIENT_ID:$CLIENT_SECRET" \
 -d "grant_type=client_credentials" | json_pp
```

> El **CLIENT_ID** y **CLIENT_SECRET** utilizados en esta petición son datos de prueba.

Como respuesta obtendrás el **access_token**:

```
{
   "expires_in" : 1507920388,
   "scope" : "profile",
   "token_type" : "Bearer",
   "access_token" : "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcmltYXJ5c2lkIjoiYTNiZTFiYzYtNDM4YS1jMzVlLWU2MDMtYjE1ZjJkMzBjZmI5IiwidW5pcXVlX25hbWUiOiJDb21lcmNpbyBkZSBQcnVlYmEiLCJncm91cHNpZCI6IkFQUEwiLCJpc3MiOiJGYWxhYmVsbGEiLCJhdWQiOiJXZWIiLCJzY29wZSI6WyJwcm9maWxlIl0sImlhdCI6MTUwNzgzMzk4OCwiZXhwIjozMDE1NzU0Mzc2fQ.FyTeDIhXhPDUy9yUq1-rDSCLhdjG2Dq3k42AcnK-PuCb6g1nfa-N52x_zI2SNmhf0TtdJcBIacfW7CXegCbU3_chcIPJJoxSkfp-GdOddzOx_loQrCwxcrlyI4tJNlGgxieNsoOXvgaC7PjfydUF23Z5GkdqK31nf5Uwc6sP38gqf-rq9CNfKLVejwEGdRPgt1YOmm87TVQax3BjVxu7GgMKzKxIa5kP72tvy0Ss3d7WMq4ZrQXWQ50a1lRa_LNwZPjJOibN-q7n8xMbHO8wXenpt2q4-U7T0cHag1IXFdZ79CfUgPMmVFKTyeY5fZDWcgKcbCpmHPbJlpKYqLI7eA"
}
```

Con el **access_token** generado, ya puedes comenzar a usar nuestras APIs para completar los pasos siguientes. 

#### 2. Crear una Intención de Captura

Luego de haber obtenido el **access_token** y con la información mínima del cliente, se debe crear una **intención de captura**.

Utilizando dicho **access_token**, debes ejecutar una llamanda a la **API de captura /captures** de la siguiente forma:

```
curl -X POST 'https://quickpay-connect-capture-card.azurewebsites.net/captures' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer ACCESS TOKEN" \
 -d '{
  "capture": "CREDIT_CARD",
  "capture_method": "TOKENIZATION",
  "cardholder": {
    "reference_id": "001389",
    "country": "CL",
    "name": "Mati 1",
    "email": "JLPrueba1@gmail.com"
  },
	"billing": {
    "line1": "Miraflores 222",
    "city": "Santiago",
    "state": "Region Metropolitana",
    "country": "CL"
  },
	
  "redirect_urls": {
    "return_url": "https://requestb.in/sfoogtsf",
    "cancel_url": "http://www.mysite.cl/cancel"
  }
}' | json_pp
```

Como respuesta obtendrás la siguiente información:

```
{
    "capture": "CREDIT_CARD",
    "capture_method": "TOKENIZATION",
    "application": "a3be1bc6-438a-c35e-e603-b15f2d30cfb9",
    "redirect_urls": {
        "return_url": "https://requestb.in/sfoogtsf",
        "cancel_url": "http://www.mysite.cl/cancel"
    },
    "billing": {
        "line1": "Miraflores 222",
        "city": "Santiago",
        "state": "Region Metropolitana",
        "country": "CL"
    },
    "cardholder": {
        "reference_id": "001389",
        "country": "CL",
        "name": "Mati 1",
        "email": "JLPrueba1@gmail.com"
    },
    "id": "f445ffde-c746-e0ce-1fc6-751f8c633800",
    "create_time": "2017-10-12T20:17:24.907Z",
    "update_time": "2017-10-12T20:17:24.907Z",
    "state": "created",
    "capture_number": "INCA-0000000046",
    "links": [
        {
            "href": "https://quickpay-connect-capture-card.azurewebsites.net/captures/f445ffde-c746-e0ce-1fc6-751f8c633800",
            "rel": "self",
            "method": "GET"
        },
        {
            "href": "https://quickpay-connect-capture-card.azurewebsites.net/captures/gateways/credit/card/f445ffde-c746-e0ce-1fc6-751f8c633800/capture",
            "rel": "capture_url",
            "method": "REDIRECT"
        }
    ]
}
```

Detalle de las URLs generadas:

+ URL 1 (**rel: self**): desde esta URL puedes consultar la información de la intención de captura.
+ URL 2 (**rel: capture_url**): desde esta URL debes acceder al formulario de captura de la tarjeta.

#### 3. Mostrar Formulario de Captura de tarjeta

Con la **capture_url** obtenida en el [paso 2](#2-crear-una-intención-de-captura) puedes desplegar el formulario de captura de tarjeta. [Puedes hacer clic aquí para ver un ejemplo de capture_url](https://quickpay-connect-capture-card.azurewebsites.net/captures/gateways/credit/card/61bf9053-dc08-15ea-7419-aba75cd3bea4/capture)

![Ejemplo de ventana Formulario](Portal02.jpg)

El cliente debe ingresar los datos solicitados en el formulario y hacer clic en **Usar esta tarjeta** para obtener el token de la tarjeta de crédito.

![Ejemplo datos a ingresar](Portal03.jpg)

La respuesta será enviada a la página de confirmacion indicada en la variable **"return_url"** de la petición a la [API de intención de Captura (paso 2)](#2-crear-una-intención-de-captura) o puedes consultar dicha respuesta llamando a la **API Revisión de Captura** de la siguiente forma:

```
 curl -X GET 'https://quickpay-connect-capture-card.azurewebsites.net/captures/{{id}}'
```

> Debes reemplazar el **id** por el obtenido en la respuesta de la **API de captura /captures** [paso 2](#2-crear-una-intención-de-captura).

**Ejemplo de respuesta enviada a la return_url:**

```
{
  "capture": "CREDIT_CARD",
  "capture_method": "TOKENIZATION",
  "application": "a3be1bc6-438a-c35e-e603-b15f2d30cfb9",
  "redirect_urls": {
    "return_url": "https://requestb.in/sfoogtsf",
    "cancel_url": "http://www.mysite.cl/cancel"
  },
  "billing": {
    "line1": "Miraflores 222",
    "city": "Santiago",
    "state": "Region Metropolitana",
    "country": "CL"
  },
  "cardholder": {
    "reference_id": "001389",
    "country": "CL",
    "name": "Mati 1",
    "email": "JLPrueba1@gmail.com"
  },
  "id": "fe5228dc-91ab-fa28-97ec-034bed089743",
  "create_time": "2017-10-12T20:29:36.035Z",
  "update_time": "2017-10-12T20:29:59.333Z",
  "state": "captured",
  "capture_number": "INCA-0000000047",
  "gateway": {
    "utf8": "âœ“",
    "req_card_number": "xxxxxxxxxxxx1111",
    "req_locale": "en",
    "signature": "2jcH1K0rV+t5UeNg4piaEEu1cHKNeLg4dqWe9SvSgUU=",
    "payment_token": "5078401995086279804103",
    "req_bill_to_surname": "1",
    "req_bill_to_address_city": "Santiago",
    "req_card_expiry_date": "12-2019",
    "reason_code": "100",
    "req_bill_to_forename": "Mati",
    "req_payment_method": "card",
    "request_token": "AhjzbwSTE0z8PAEIwQTHigFP6t6n3RCYAOOsMmkmW6QHAX2gJgAA+A6E",
    "req_amount": "0",
    "req_bill_to_email": "JLPrueba1@gmail.com",
    "transaction_id": "5078401995086279804103",
    "req_currency": "CLP",
    "req_card_type": "001",
    "decision": "ACCEPT",
    "req_merchant_defined_data1": "0e0b75dc-2cb1-81f5-9e19-054741744455",
    "message": "Request was processed successfully.",
    "signed_field_names": "transaction_id,decision,req_access_key,req_profile_id,req_transaction_uuid,req_transaction_type,req_reference_number,req_amount,req_currency,req_locale,req_payment_method,req_bill_to_forename,req_bill_to_surname,req_bill_to_email,req_bill_to_address_line1,req_bill_to_address_city,req_bill_to_address_state,req_bill_to_address_country,req_card_number,req_card_type,req_card_expiry_date,req_merchant_defined_data1,message,reason_code,request_token,payment_token,signed_field_names,signed_date_time",
    "req_transaction_uuid": "INCA-0000000047",
    "req_bill_to_address_country": "CL",
    "req_transaction_type": "create_payment_token",
    "req_access_key": "184e3533f6583a178b5948606a5e2b49",
    "req_profile_id": "808DE8FD-E2EB-4350-AFA7-6F4508734F53",
    "req_reference_number": "001389",
    "req_bill_to_address_state": "Region Metropolitana",
    "signed_date_time": "2017-10-12T20:29:59Z",
    "req_bill_to_address_line1": "Miraflores 222",
    "panLast4": "1111"
  }
}
```

El **id** generado corresponde al **Token de la tarjeta**.

> **id de ejemplo**"id": "fe5228dc-91ab-fa28-97ec-034bed089743"

#### 4. Intención de Pago

Para completar el pago con el **token de la tarjeta** debes ingresar el **id** obtenido previamente de la **return_url** en el campo **capture_token** de la petición a la API de **Intención de Pago /payments**, el **access_token** generado en el [paso 1](#1-obtener-un-token-de-acceso) y hacer el llamado de la siguiente forma:

```
curl -X POST 'https://quickpay-connect-checkout.azurewebsites.net/payments' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer access_token" \
 -d '{ 
   "intent": "sale", 
   "payer": { 
     "payer_info": { 
       "email": "jlprueba1@quickpay.com", 
       "full_name": "JL Prueba 1",
       "country": "CL",
       "documentNumber": "123123123",
       "documentType": "RUT"
     }, 
     "payment_method": "QUICKPAY_TOKEN",
     "capture_token": "fe5228dc-91ab-fa28-97ec-034bed089743"
   }, 
   "transaction": { 
     "reference_id": "OD0000233", 
     "description": "Transaction detailed description", 
     "soft_descriptor": "Short Description", 
     "amount": { 
       "currency": "CLP", 
       "total": 4500, 
       "details": { 
         "subtotal": 810, 
         "tax": 190, 
         "shipping": 0, 
         "shipping_discount": 0 
       } 
     }, 
     "item_list": { 
       "shipping_address": { 
         "line1": "Miraflores 222", 
         "city": "Santiago", 
         "country_code": "CL", 
         "phone": "+56 9 1234 5674", 
         "type": "HOME_OR_WORK", 
         "recipient_name": "JL Prueba 1" 
       }, 
       "shipping_method": "DIGITAL", 
       "items": [ 
         { 
           "sku": "1231232", 
           "name": "Destornillador 2344", 
           "description": "Destornillador SCL - ONT", 
           "quantity": 1, 
           "price": 500, 
           "tax": 0 
         } 
       ] 
     } 
   }, 
   "redirect_urls": { 
     "return_url": "https://requestb.in/sfoogtsf", 
     "cancel_url": "https://chao.com" 
   } 
 }' | json_pp
```

Como respuesta obtendrás la siguiente información:

```
{
    "intent": "sale",
    "application": "a3be1bc6-438a-c35e-e603-b15f2d30cfb9",
    "redirect_urls": {
        "return_url": "https://requestb.in/sfoogtsf",
        "cancel_url": "https://chao.com"
    },
    "transaction": {
        "reference_id": "OD0000233",
        "description": "Transaction detailed description",
        "soft_descriptor": "Short Description",
        "item_list": {
            "shipping_method": "DIGITAL",
            "items": [
                {
                    "sku": "1231232",
                    "name": "Destornillador 2344",
                    "description": "Destornillador SCL - ONT",
                    "quantity": 1,
                    "price": 500,
                    "tax": 0
                }
            ],
            "shipping_address": {
                "line1": "Miraflores 222",
                "city": "Santiago",
                "country_code": "CL",
                "phone": "+56 9 1234 5674",
                "type": "HOME_OR_WORK",
                "recipient_name": "JL Prueba 1"
            }
        },
        "amount": {
            "currency": "CLP",
            "total": 4500,
            "details": {
                "subtotal": 810,
                "tax": 190,
                "shipping": 0,
                "shipping_discount": 0
            }
        }
    },
    "payer": {
        "payer_info": {
            "documentType": "RUT",
            "documentNumber": "123123123",
            "country": "CL",
            "full_name": "JL Prueba 1",
            "email": "jlprueba1@quickpay.com"
        },
        "payment_method": "QUICKPAY_TOKEN",
        "capture_token": "0e0b75dc-2cb1-81f5-9e19-054741744455"
    },
    "links": [
        {
            "href": "https://quickpay-connect-checkout.azurewebsites.net/payments/0fdcd938-62c7-aab2-5048-c2f172d495ac",
            "rel": "self",
            "method": "GET"
        },
        {
            "href": "https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/0fdcd938-62c7-aab2-5048-c2f172d495ac/pay",
            "rel": "approval_url",
            "method": "REDIRECT"
        },
        {
            "href": "https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/0fdcd938-62c7-aab2-5048-c2f172d495ac/reverse",
            "rel": "reverse_method",
            "method": "POST"
        },
        {
            "href": "https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/0fdcd938-62c7-aab2-5048-c2f172d495ac/silent",
            "rel": "silent_charge",
            "method": "POST"
        }
    ],
    "id": "0fdcd938-62c7-aab2-5048-c2f172d495ac",
    "create_time": "2017-10-12T20:37:48.390Z",
    "update_time": "2017-10-12T20:37:48.390Z",
    "state": "created",
    "invoice_number": "INPA-0000000093"
}
```

Obtendrás los Links:

- **self**: desde esta URL puedes consultar la información de la captura.
- **approval_url**: desde esta URL el cliente debe autorizar el pago.
- **reverse_method**: para anular la transacción, debes hacer el llamado a este endpoint desde la [API de Anulación](Anulaciones.md).
- **silent_charge**: llamando a este endpoint desde la [API silent_charge](#52-silent-charge) puedes ejecutar el cargo a la tarjeta de cŕedito del cliente sin pasar por la intención de pago.

#### 5. ¿Cómo realizar el cargo a la tarjeta?

Te ofrecemos dos opciones para realizar el cargo a la tarjeta del cliente, a continuación podrás ver el detalle de cada una: 

##### 5.1 Approval

Si quieres utilizar esta opción, necesitas que el cliente apruebe el pago para ejecutar el cargo a la tarjeta de crédito. Para ello debes desplegar la ventana de aprobación del pago a partir de la [approval_url](https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/0fdcd938-62c7-aab2-5048-c2f172d495ac/pay) obtenida en el [paso 3](#3-mostrar-formulario-de-captura-de-tarjeta).

![Ejemplo Approval](Portal04.jpg)

Esta ventana permite dos opciones al cliente:

* Aprobar Pago y ejecutar el cargo a la tajeta de crédito
* Cancelar

###### 5.1.1 Aprobar Pago y ejecutar el cargo a la tajeta de crédito

  Si recibes una respuesta con **"state": "paid"** en la URL indicada como **return_url** o consultando desde la **URL self**, entonces el cliente ha aprobado del pago y se ejecuta el cargo a la tajeta de crédito.

  ```
  {
      "intent": "sale",
      "application": "a3be1bc6-438a-c35e-e603-b15f2d30cfb9",
      "redirect_urls": {
          "return_url": "https://requestb.in/sfoogtsf",
          "cancel_url": "https://chao.com"
      },
      "transaction": {
          "reference_id": "OD0000233",
          "description": "Transaction detailed description",
          "soft_descriptor": "Short Description",
          "item_list": {
              "shipping_method": "DIGITAL",
              "items": [
                  {
                      "sku": "1231232",
                      "name": "Destornillador 2344",
                      "description": "Destornillador SCL - ONT",
                      "quantity": 1,
                      "price": 500,
                      "tax": 0,
                      "_id": "59e0c580f23884b8cc3aa5c5"
                  }
              ],
              "shipping_address": {
                  "line1": "Miraflores 222",
                  "city": "Santiago",
                  "country_code": "CL",
                  "phone": "+56 9 1234 5674",
                  "type": "HOME_OR_WORK",
                  "recipient_name": "JL Prueba 1"
              }
          },
          "amount": {
              "currency": "CLP",
              "total": 4500,
              "details": {
                  "subtotal": 810,
                  "tax": 190,
                  "shipping": 0,
                  "shipping_discount": 0
              }
          }
      },
      "payer": {
          "payer_info": {
              "documentType": "RUT",
              "documentNumber": "123123123",
              "country": "CL",
              "full_name": "JL Prueba 1",
              "email": "jlprueba1@quickpay.com"
          },
          "payment_method": "QUICKPAY_TOKEN",
          "capture_token": "cda62314-1258-ff27-6a8b-6323955998ce"
      },
      "links": [],
      "id": "79aef545-3dda-dfae-e924-cd1c60eebdd1",
      "create_time": "2017-10-13T13:54:09.156Z",
      "update_time": "2017-10-13T13:54:36.169Z",
      "state": "paid",
      "invoice_number": "INPA-0000000096",
      "gateway": {
          "installments_number": "1",
          "merchantReferenceCode": "INPA-0000000096",
          "requestID": "5079028762296913904009",
          "decision": "ACCEPT",
          "reasonCode": "100",
          "requestToken": "Ahj7/wSTE1Wu9XYbdkGJik/oE2bCnAU/oE2bCnJgbGUcdoZNJMt0gOAvtDAnJiarXersNuyDEgAAahIC",
          "purchaseTotals": {
              "currency": "CLP"
          },
          "ccAuthReply": {
              "reasonCode": "100",
              "amount": "4500",
              "authorizationCode": "570110",
              "avsCode": "1",
              "authorizedDateTime": "2017-10-13T13:54:36Z",
              "processorResponse": "1",
              "paymentNetworkTransactionID": "111222",
              "ownerMerchantID": "falabella",
              "processorTransactionID": "cca1703e27f34fc3bd661c3aaaa080a6"
          },
          "ccCaptureReply": {
              "reasonCode": "100",
              "requestDateTime": "2017-10-13T13:54:36Z",
              "amount": "4500"
          },
          "additionalProcessorResponse": "9d7549aa-3207-4ee3-aff8-f17330a0c4c1",
          "capture_token": "cda62314-1258-ff27-6a8b-6323955998ce"
      }
  }
  ```

###### 5.1.2 Cancelar

  Si recibes una respuesta con **"state": "canceled"** en la URL indicada como **cancel_url** o consultando desde la **URL self**, entonces el cliente ha cancelado el pago.

```
{
  "intent": "sale",
  "application": "a3be1bc6-438a-c35e-e603-b15f2d30cfb9",
  "redirect_urls": {
    "return_url": "https://requestb.in/sfoogtsf",
    "cancel_url": "https://chao.com"
  },
  "transaction": {
    "reference_id": "OD0000233",
    "description": "Transaction detailed description",
    "soft_descriptor": "Short Description",
    "item_list": {
      "shipping_method": "DIGITAL",
      "items": [
        {
          "sku": "1231232",
          "name": "Destornillador 2344",
          "description": "Destornillador SCL - ONT",
          "quantity": 1,
          "price": 500,
          "tax": 0,
          "_id": "59e0b597f23884b8cc3aa5c3"
        }
      ],
      "shipping_address": {
        "line1": "Miraflores 222",
        "city": "Santiago",
        "country_code": "CL",
        "phone": "+56 9 1234 5674",
        "type": "HOME_OR_WORK",
        "recipient_name": "JL Prueba 1"
      }
    },
    "amount": {
      "currency": "CLP",
      "total": 4500,
      "details": {
        "subtotal": 810,
        "tax": 190,
        "shipping": 0,
        "shipping_discount": 0
      }
    }
  },
  "payer": {
    "payer_info": {
      "documentType": "RUT",
      "documentNumber": "123123123",
      "country": "CL",
      "full_name": "JL Prueba 1",
      "email": "jlprueba1@quickpay.com"
    },
    "payment_method": "QUICKPAY_TOKEN",
    "capture_token": "d246ba70-6340-98a3-fbd5-b243a61314a9"
  },
  "links": [
    
  ],
  "id": "9ece98a3-9c64-6f0d-ee12-39e68745017d",
  "create_time": "2017-10-13T12:46:16.497Z",
  "update_time": "2017-10-13T12:47:00.733Z",
  "state": "canceled",
  "invoice_number": "INPA-0000000095",
  "gateway": {
    "message": "user cancel error",
    "amount": 4500,
    "capture_token": "d246ba70-6340-98a3-fbd5-b243a61314a9"
  }
}
```

##### 5.2 Silent Charge

Con esta opción, no es requerida la aprobación del cliente para ejecutar el cargo a la tarjeta de crédito.

Necesitas el **access_token** obtenido en la **Autenticación** y el **id (Token de la tarjeta)** generado en la **Intención de captura**, para ejecutar una llamada a la **API de Silent Charge /silent** de la siguiente forma:

```
 curl -v -X POST 'https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/{Token_de_tarjeta}/silent' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer access_token"
```
[Volver al menú anterior](Pasarela-de-pagos.md)
