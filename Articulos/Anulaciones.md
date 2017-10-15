# Anulaciones

Puedes anular tus transacciones a través de distintos canales:

- **API's**

Utilizando el **access_token** obtenido en la **Autenticación** y el **id (Token de la tarjeta)** generado en la **Intención de captura**, debes ejecutar una petición a la **API de Anulación /reverse** de la siguiente forma:

```
 curl -v -X POST 'https://quickpay-connect-checkout.azurewebsites.net/payments/gateways/quickpay/token/{Token_de_tarjeta}/reverse' \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer access_token"
```
- **Portal de Comercio**

  > PROXIMAMENTE

- **Portal de Transbank**

  Puedes obtener mayor información en <https://www.transbank.cl/web/landing/2017/anulaciones>
