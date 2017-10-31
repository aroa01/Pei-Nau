# Estrategia de Funcionamiento

La estrategia en la cual se basa este componente, se encuentra bajo el concepto de [Webhook][webhook], que permite a un [Microservicio][microservice] eliminar el nivel de acoplamiento fuerte contra otros servicios / microservicios permitiendo seguir manteniendo su resilencia y [bajo acoplamiento][loose-coupling], inclusive cuando alguna pieza externa de funcionamiento critico se haya detenido por algúna falla en su funcionamiento.

Para lograr lo anterior, es necesario que el sistema de captura (tokenizador) implemente un sistema de eventos en que el componente **intermediate proxy pull** pueda subcribirse, y en el cual a su vez bajo el mismo concepto previamente explicado, permita que los componentes fuera de alcance PCI, puedan subscribirse para la comunicación de los datos no sensibles de captura a sus fuentes persistentes.

Resumiendo lo anterior, para el correcto funcionamiento de este componente intermediario en el ecosistema de **Quickpay Connect** es necesaria las siguientes configuraciones a nivel interno.

- **Subscripción** al evento de captura exitosa "card_captured", indicando la URL a la cual la información de captura (*datos no sensibles*) será enviada (pull). 
> **Nota:**
> En el contexto de  *Quickpay Connect*, es necesario configurar la URL base de recepción del componente API *intermediate proxy pull* a:  **{api-intermediate-proxy-pull}/captures/tokenization/webhook_callback** 

- **Subcripción** al evento de *intermediate proxy pull*, indicando la URL a la cual la información de captura, será enviada (pull).
> **Nota:**
> En el contexto de *Quickpay Connect*, es necesario configurar la URL base de recepcion del componente API *Checkout* a: **{api-checkout}/captures/tokenization/webhook_callback**.

  ![diagrama de secuencia][secuence-diagram]
  ->*Diagrama de secuencias*<-

---
## *Estructura de los datos:* Tokenizador -> Intermediate proxy pull

````bash
curl -v -X POST 'https://{api-capture-card}/captures/tokenization/webhook_callback' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer Your-Valid-Access-Token" \
 -d '{
  "capture": "CREDIT_CARD",
  "capture_method": "TOKENIZATION",
  "application": "acb361cc-e4e7-24fb-d4e2-17bb3aa74066",
  "cardholder": {
    "reference_id": "Merchant_id_reference",
    "name": "Jhon Doe",
    "email": "jhondoe@gmail.com",
    "country": "CL"
  },
  "billing": {
    "line1": "Miraflores 222",
    "city": "Santiago",
    "state": "Region Metropolitana",
    "country": "CL"
  },
  "id": "71230785-c66e-4c96-b2d9-658bfd41235b",
  "create_time": "2017-09-01T04:41:22.220Z",
  "update_time": "2017-09-01T04:41:31.458Z",
  "state": "captured",
  "capture_number": "INCA-0000000432",
  "gateway": {
    "utf8": "✓",
    "req_card_number": "xxxxxxxxxxxx1111",
    "payment_token": "5042408910336785404106",
    "req_bill_to_surname": "Doe",
    "req_bill_to_address_city": "Santiago",
    "req_card_expiry_date": "02-2018",
    "reason_code": "100",
    "req_bill_to_forename": "Jhon",
    "req_payment_method": "card",
    "request_token": "AhjzbwSTEVl7HB+4nFDKigFP5x2KJkiYAOJcMmkmW6QHAX2gJgAAJQ9A",
    "req_amount": "0",
    "req_bill_to_email": "jhondoe@gmail.com",
    "transaction_id": "5042408910336785404106",
    "req_currency": "CLP",
    "req_card_type": "001",
    "decision": "ACCEPT",
    "message": "Request was processed successfully.",
    "req_transaction_uuid": "INCA-0000000432",
    "req_bill_to_address_country": "CL",
    "req_profile_id": "5D5F2A29-2428-4E8B-BFCE-D55043704C90",
    "req_reference_number": "Merchant_id_reference",
    "req_bill_to_address_state": "Region Metropolitana",
    "signed_date_time": "2017-09-01T04:41:31Z",
    "req_bill_to_address_line1": "Miraflores 222",
    "panLast4": "1111"
  }
}'
````

| Nombre        | Tipo            | Descripción  | Requerido |
| ------------- | --------------- | ------------ | --------- |
| id            | String (Guid)   | Identificador único de la intención              | Sí |
| capture | Enum [:arrow_upper_right:][enumeration-capture] | Tipo de Captura establecida en la intención. | Sí |
| capture_method| Enum [:arrow_upper_right:][enumeration-capture-method] | Método de Captura que se usará  | Sí |
| application   | String (Guid)   | Identificador de la aplicación del comercio quien solicito la captura | Sí |
| cardholder | Object        | Datos del tarjeta habiente | Sí |
| cardholder.reference_id | String        | Identificador externo del comercio | No |
| cardholder.country | String ([ISO 3166-1 alpha-2][country-iso]) | País de residencia del tarjeta habiente | Sí|
| cardholder.name | String      | Nombre del tarjeta habiente tal como se muestra en la tarjeta | Sí |
| billing | String      | Dirección de facturación asociada a la tarjeta  | Sí |
| billing.line1 | String      | Dirección de facturación | Sí |
| billing.line2 | String      | Dirección 2 de facturación | No |
| billing.city | String      | Ciudad para facturación | Sí |
| billing.state | String      | Comuna de facturación | Sí |
| billing.country | String ([ISO 3166-1 alpha-2][country-iso])     | País de facturación| Sí |
| create_time | String ([ISO 8601][datetime-iso]) | Fecha de creación de la intención | Sí |
| update_time | String ([ISO 8601][datetime-iso]) | Fecha de actualización de la intención | Sí |
| state | Enum [:arrow_upper_right:][enumeration-state] | Estado actual de la intención | Sí |
| capture_number | String (Correlativo) | Identificador legible de la intención | Sí |
| redirect_urls | Object      | Url de redirección dependiendo del estado de la captura una vez finalizado el proceso de captura | Sí |
| redirect_urls.return_url | String (Url)      | Url de redirección al producirse una captura exitosa | Sí |
| redirect_urls.cancel_url | String (Url)      | Url de redirección al producirse una captura fallida | Sí |
| gateway | Object      | Datos de la captura (varía dependiendo del metodo de captura establecido ) | Sí |

## *Estructura de los datos:* Intermediate proxy pull -> Checkout

````bash
curl -v -X POST 'https://{api-checkout}/captures/tokenization/webhook_callback' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer Your-Valid-Access-Token" \
 -d '{
  "capture": "CREDIT_CARD",
  "capture_method": "TOKENIZATION",
  "application": "acb361cc-e4e7-24fb-d4e2-17bb3aa74066",
  "cardholder": {
    "reference_id": "Merchant_id_reference",
    "name": "Jhon Doe",
    "email": "jhondoe@gmail.com",
    "country": "CL"
  },
  "billing": {
    "line1": "Miraflores 222",
    "city": "Santiago",
    "state": "Region Metropolitana",
    "country": "CL"
  },
  "id": "71230785-c66e-4c96-b2d9-658bfd41235b",
  "create_time": "2017-09-01T04:41:22.220Z",
  "update_time": "2017-09-01T04:41:31.458Z",
  "state": "captured",
  "capture_number": "INCA-0000000432",
  "gateway": {
    "utf8": "✓",
    "req_card_number": "xxxxxxxxxxxx1111",
    "payment_token": "5042408910336785404106",
    "req_bill_to_surname": "Doe",
    "req_bill_to_address_city": "Santiago",
    "req_card_expiry_date": "02-2018",
    "reason_code": "100",
    "req_bill_to_forename": "Jhon",
    "req_payment_method": "card",
    "request_token": "AhjzbwSTEVl7HB+4nFDKigFP5x2KJkiYAOJcMmkmW6QHAX2gJgAAJQ9A",
    "req_amount": "0",
    "req_bill_to_email": "jhondoe@gmail.com",
    "transaction_id": "5042408910336785404106",
    "req_currency": "CLP",
    "req_card_type": "001",
    "decision": "ACCEPT",
    "message": "Request was processed successfully.",
    "req_transaction_uuid": "INCA-0000000432",
    "req_bill_to_address_country": "CL",
    "req_profile_id": "5D5F2A29-2428-4E8B-BFCE-D55043704C90",
    "req_reference_number": "Merchant_id_reference",
    "req_bill_to_address_state": "Region Metropolitana",
    "signed_date_time": "2017-09-01T04:41:31Z",
    "req_bill_to_address_line1": "Miraflores 222",
    "panLast4": "1111"
  }
}'
````

| Nombre        | Tipo            | Descripción  | Requerido |
| ------------- | --------------- | ------------ | --------- |
| id            | String (Guid)   | Identificador único de la intención              | Sí |
| capture | Enum [:arrow_upper_right:][enumeration-capture] | Tipo de Captura establecida en la intención. | Sí |
| capture_method| Enum [:arrow_upper_right:][enumeration-capture-method] | Método de Captura que se usará  | Sí |
| application   | String (Guid)   | Identificador de la aplicación del comercio quien solicito la captura | Sí |
| cardholder | Object        | Datos del tarjeta habiente | Sí |
| cardholder.reference_id | String        | Identificador externo del comercio | No |
| cardholder.country | String ([ISO 3166-1 alpha-2][country-iso]) | País de residencia del tarjeta habiente | Sí|
| cardholder.name | String      | Nombre del tarjeta habiente tal como se muestra en la tarjeta | Sí |
| billing | String      | Dirección de facturación asociada a la tarjeta  | Sí |
| billing.line1 | String      | Dirección de facturación | Sí |
| billing.line2 | String      | Dirección 2 de facturación | No |
| billing.city | String      | Ciudad para facturación | Sí |
| billing.state | String      | Comuna de facturación | Sí |
| billing.country | String ([ISO 3166-1 alpha-2][country-iso])     | País de facturación| Sí |
| create_time | String ([ISO 8601][datetime-iso]) | Fecha de creación de la intención | Sí |
| update_time | String ([ISO 8601][datetime-iso]) | Fecha de actualización de la intención | Sí |
| state | Enum [:arrow_upper_right:][enumeration-state] | Estado actual de la intención | Sí |
| capture_number | String (Correlativo) | Identificador legible de la intención | Sí |
| redirect_urls | Object      | Url de redirección dependiendo del estado de la captura una vez finalizado el proceso de captura | Sí |
| redirect_urls.return_url | String (Url)      | Url de redirección al producirse una captura exitosa | Sí |
| redirect_urls.cancel_url | String (Url)      | Url de redirección al producirse una captura fallida | Sí |
| gateway | Object      | Datos de la captura (varía dependiendo del metodo de captura establecido ) | Sí |

# TLDR:

 **¿Porqué es necesario ejecutar dos envios de información (*tokenizador -> intermediate proxy pull -> checkout*) para finalmente enviar la información al componente checkout, cúando podriamos hacerlo directamente desde el tokenizador -> checkout?**
 
 Solo por el hecho de sacar del alcance PCI a la plataforma checkout, y que esta pueda estar fuera de las altas restricciones de los ambientes PCI, ya que cómo bien se plantea, facilmente podria ser una sola llamada, directamente del tokenizdor -> checkout, pero esta definición produciría que checkout estuviera dentro del alcance PCI; y  es por ello que el planteamiento de envio de la información sea tokenizador -> intermediate proxy pull -> checkout.

---

**¿Qué pasa si al momento de captura de una tarjeta la conexión entre el tokenizador y el intermediate proxy pull se pierde?**
 
Un sistema resiliente son aquellos sistemas que estan pensados en las fallas, y que por ende, aunque un componente externo falle, la solicitud de envió quedara encolada para ser re-intentada mas tarde, por lo que si por algún problema interno la malla de conexión entre ambos componentes encapsulados en la infraestructura PCI falla, el sistema de tokenizador , dejará encolada el envio de información para un tiempo mas adelante, hasta que el código de respuesta HTTP entregado por el sistema subscrito (intermediate proxy pull), sea un código HTTP exitoso (200,201,203,204, etc).

Mas Información: [The Reactive Manifesto][the-reactive-manifesto]

---

[secuence-diagram]: images/how-it-work-1.png

[the-reactive-manifesto]: http://www.reactivemanifesto.org/es
[webhook]: https://en.wikipedia.org/wiki/Webhook
[microservice]: https://es.wikipedia.org/wiki/Arquitectura_de_microservicios
[loose-coupling]: https://en.wikipedia.org/wiki/Loose_coupling

[datetime-iso]: https://es.wikipedia.org/wiki/ISO_8601
[country-iso]: https://es.wikipedia.org/wiki/ISO_3166-1#Tabla_de_c.C3.B3digos_alfa-2_asignados_o_reservados
[enumeration-state]: ../product-capture-card/enumeration-state.md
[enumeration-capture]: ../product-capture-card/enumeration-capture.md
[enumeration-capture-method]: ../product-capture-card/enumeration-capture-method.md
