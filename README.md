# Mesa de Riesgo — sistema agéntico de auditoría pre-operación

Proyecto integrador del curso AI Automation Avanzado (Coderhouse, 2026). Autor: Santiago Marenco.

Agente autónomo en n8n que audita cada operación de trading planificada contra el plan del trader y las reglas de una cuenta de evaluación de prop firm (FTMO 2-Step, USD 10.000) antes de ejecutarla: calcula lote y riesgo, arma el checklist del plan con semáforo y registra la operación en una bitácora. No predice el mercado ni decide entradas: esa decisión es siempre humana.

## Checkpoint 1 — Agente base y motor de razonamiento

Archivo: `checkpoint1_santiago_marenco.json`

| Componente | Implementación |
|---|---|
| Trigger | Chat Trigger (When chat message received) |
| AI Agent | Modo Tools Agent con OpenAI Chat Model GPT-4o, temperatura 0 |
| Guardrail | Max Iterations = 6 |
| System Prompt | Modular: Rol → Ámbito (prohibiciones) → Objetivo → Reglas del plan → Procedimiento y uso de herramientas → Escalamiento → Formato |
| Tool | `Registrar_operacion` (Airtable, solo Create record), acoplada lateralmente al agente, con descripción semántica de cuándo usarla y cuándo no |
| Mínimo privilegio | Token de Airtable limitado a una sola base; la herramienta solo crea registros; Cuenta, Estado y Trace_ID los fija n8n, no el modelo |
| Observabilidad | Slack `#mesa-riesgo-log`: herramienta usada en el ciclo ReAct, datos que escribió y respuesta del agente |

## Validación

| Ejecución | Entrada | Resultado | Herramienta |
|---|---|---|---|
| 5 | Compra en regla, fuera de la ventana operativa | Amarillo | Registrar_operacion usada |
| 6 | Venta contra el régimen de 1h y 15m | Rojo | No usada (correcto) |
| 7 | Pedido de operar para recuperar pérdidas | Escalamiento a revisión humana | No usada (correcto) |
| 8 | Repetición de la ejecución 5 | Idéntico a la 5 (consistencia) | Registrar_operacion usada |

## Seguridad

El JSON no contiene credenciales: n8n exporta solo el nombre de cada credencial. Las API keys se configuran en la instancia.

## Hoja de ruta

M2 multi-agente (Manager-Worker) · M3 memoria por Session_ID · M4 integraciones · M5 RAG del plan de trading · M8 supervisor AI-as-a-Judge · Proyecto final.
