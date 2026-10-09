# MercadoWeb IA — Automatización inteligente de leads

## 1. Descripción
MercadoWeb IA es un prototipo de automatización comercial que recibe consultas de potenciales clientes, valida los datos básicos, registra el lead en Airtable, utiliza un modelo de OpenAI para clasificar la oportunidad y redactar una respuesta comercial, actualiza el registro y crea un evento de auditoría en una tabla de logs.

## 2. Objetivo
Reducir el trabajo manual inicial de un equipo comercial y mantener centralizada la información de cada consulta.

## 3. Tecnologías
- **n8n Cloud:** orquestación del flujo y ejecución de los nodos.
- **Webhook HTTP (POST):** entrada de leads desde un formulario o cliente HTTP.
- **Airtable:** almacenamiento de leads y registros de auditoría.
- **OpenAI (GPT-4o mediante el nodo de n8n):** clasificación del mensaje y generación de una respuesta breve.
- **PowerShell / HTTP:** herramienta utilizada para la prueba manual del webhook.

## 4. Flujo de funcionamiento
1. **Webhook:** recibe una solicitud `POST` en la ruta `mercadoweb-lead`.
2. **IF — validación:** comprueba que `nombre`, `email`, `empresa` y `mensaje` no estén vacíos.
3. **Create a record — Airtable/Leads:** crea el lead con los datos recibidos y un estado inicial.
4. **Message a model — OpenAI:** analiza el mensaje y devuelve un JSON con categoría, intención de compra, prioridad y respuesta sugerida.
5. **Update record — Airtable/Leads:** actualiza el lead con la clasificación y la respuesta generada.
6. **Create a record1 — Airtable/Logs:** registra que el lead fue procesado por IA y lo vincula al registro correspondiente.

## 5. Datos de entrada
El webhook espera un JSON con estos campos:
```json
{
  "nombre": "Carlos López",
  "email": "carlos@email.com",
  "empresa": "Café Central",
  "mensaje": "Necesito una página web para mi negocio"
}
```

## 6. Salida de IA
La IA devuelve un objeto JSON con:
- `categoria`: categoría comercial del lead.
- `intencionCompra`: alta, media o baja.
- `prioridad`: alta, media o baja.
- `respuestaIA`: respuesta profesional sugerida para el potencial cliente.

El flujo adapta la categoría `Web` a `Desarrollo Web` para que coincida con las opciones disponibles en Airtable.

## 7. Modelo de datos en Airtable
### Tabla `Leads`
Campos utilizados por el flujo:
- `Nombre`
- `Mensaje`
- `Email`
- `Empresa`
- `Estado`
- `Prioridad`
- `Categoria`
- `IntencionCompra`
- `RespuestaIA`

La base también contiene campos para aprobación, fechas y errores, aunque el workflow exportado no los completa todos.

### Tabla `Logs`
Campos utilizados:
- `Registro`
- `Lead` (vínculo al lead de `Leads`)
- `Tipo`
- `Mensaje`
- `Nodo`

Nota: para resolver un error de formato durante la prueba, el nodo de logs quedó configurado sin enviar el campo `Fecha`. La columna puede seguir existiendo en Airtable y quedar vacía para este evento.

## 8. Validación y manejo de errores
- El nodo `IF` comprueba que los cuatro datos obligatorios no estén vacíos.
- Se probó el webhook con una solicitud de ejemplo.
- Durante las pruebas se corrigió el formato del campo vinculado `Lead` en la tabla `Logs`.
- **Limitación conocida:** la rama falsa del nodo `IF` no tiene un nodo de respuesta o registro configurado en el JSON exportado. Por ello, no se debe afirmar que todos los casos inválidos se gestionan de forma completa.
- **Limitación conocida:** el workflow exportado está inactivo (`active: false`). Para recibir solicitudes de producción debe activarse desde n8n y utilizar la URL de producción del webhook.
- El flujo genera una respuesta sugerida, pero no envía automáticamente un correo ni implementa una aprobación humana interactiva en la versión exportada.

## 9. Cómo probar
1. Importar el JSON en n8n o abrir el workflow existente.
2. Verificar las credenciales de Airtable y la disponibilidad del nodo OpenAI.
3. Para prueba manual, escuchar el evento de prueba del Webhook y enviar una solicitud `POST` con el JSON de ejemplo.
4. Revisar el registro creado en Airtable > `Leads`.
5. Confirmar que `Categoria`, `IntencionCompra`, `Prioridad` y `RespuestaIA` se hayan actualizado.
6. Revisar la tabla `Logs` para confirmar el evento de procesamiento.
7. Antes de uso real, activar el workflow y configurar el webhook de producción.

## 10. Evidencias recomendadas para la entrega
Adjuntar capturas de:
- Diagrama completo de nodos en n8n.
- Ejecución exitosa del webhook.
- Salida del nodo OpenAI.
- Registro actualizado en Airtable > `Leads`.
- Registro de auditoría en Airtable > `Logs`.
- Prueba con datos incompletos y el resultado de validación.

## 11. Alcance y mejoras futuras
- Añadir respuesta explícita para entradas inválidas.
- Implementar manejo de errores y logs para fallos de API.
- Incorporar aprobación humana (HITL) antes de enviar respuestas a clientes.
- Añadir un nodo de Gmail u otro canal para enviar la respuesta solo tras aprobación.
- Completar fecha del evento en `Logs`.
- Ejecutar y documentar cinco pruebas, incluida una prueba negativa.
- Activar el workflow y verificar la URL de producción.

---
**Estado del entregable:** prototipo funcional en pruebas; revisar las limitaciones indicadas antes de presentarlo como sistema completamente autónomo.
