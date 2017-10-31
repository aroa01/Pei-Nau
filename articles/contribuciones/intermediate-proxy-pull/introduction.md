# Intermediate Proxy Pull (PCI CDE)

Intermediate Proxy Pull (*PCI CDE*) nos permite, restringir el **alcance PCI** del modulo capture card a solo su ámbito (tokenización de tarjetas bancarias), dejando afuera de este alcance al ecosistema con que hoy cuenta la plataforma de pagos **Quickpay Connect**.

El componente tiene por finalidad, ser un **intermediario** entre los componentes bajo el alcance PCI , y el resto de los componentes que hoy componen al ecosistema.

Su forma de funcionar se basa en el **"empuje" (pull)** de información no sensible hacia el componente encargado de realizar los cargos bancarios a través de los medios de pagos disponibles (*API Checkout*) bajo mensajes HTTP ([Webhook])https://en.wikipedia.org/wiki/Webhook), empujando la información mínima necesaria para ejecutar un cargo bancario a la tarjeta del tarjeta habiente (información basica del cliente y el token de la tarjeta capturada).

[Ver guía de funcionamiento](how-it-work.md)
