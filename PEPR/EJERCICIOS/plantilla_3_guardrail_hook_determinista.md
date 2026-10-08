# Configuración del Hook y Script Determinista (Plantilla)

## 1. Registro en el runtime del Agente (`hooks.json`)

```json
{
  "hooks": {
    "preToolUse": [
      {
        "matcher": "^query_siem_historical_traffic$",
        "command": "python3 guardrail_siem.py"
      }
    ]
  }
}
```

## 2. Script Interceptor (Esqueleto en Python)

Este script debe leer los datos que el orquestador le pasa (por entrada estándar) y aplicar la lógica determinista para permitir o bloquear la acción.

```python
import sys
import json
import ipaddress

def aplicar_logica_de_negocio(datos_entrada):
    """
    Validación determinista y estricta (Fail-closed).
    Aplica el principio de lista blanca para validar que los parámetros
    no contienen inyecciones de comandos (Bash, KQL, SPL).
    Retorna True si es seguro, False si se debe bloquear.
    """
    es_seguro = True 
    
    ip_str = datos_entrada.get("ip_origen", "")
    timeframe_str = datos_entrada.get("timeframe", "")
    
    # 1. Validación estricta de la IP
    # Si la cadena contiene comandos como "1.1.1.1; drop table" o "$(whoami)", 
    # la librería ipaddress lanzará ValueError y bloquearemos la acción.
    try:
        ipaddress.ip_address(ip_str.strip())
    except ValueError:
        es_seguro = False
        
    # 2. Validación estricta del timeframe mediante Lista Blanca
    # Bloquea intentos de evasión que intenten alterar el rango de la consulta
    timeframes_permitidos = {"15m", "30m", "1h", "6h", "12h", "24h"}
    if timeframe_str.strip().lower() not in timeframes_permitidos:
        es_seguro = False
        
    return es_seguro

def main():
    try:
        # 1. Leer el payload JSON del entorno (orquestador/MCP)
        input_data = json.load(sys.stdin)
        
        # 2. Extraer los argumentos que necesitamos validar
        # Extraemos el diccionario completo de parámetros de la herramienta
        argumentos_a_validar = input_data.get("arguments", {})
        
    except Exception:
        print("Error: Payload mal formado.")
        sys.exit(1) # exit distinto de cero bloquea el flujo

    # 3. Pasar por el guardrail determinista
    if not aplicar_logica_de_negocio(argumentos_a_validar):
        print(f"GUARDRAIL TRIGGERED: Validación fallida para el parámetro. Posible inyección de comandos en el SIEM.")
        sys.exit(1) # Bloquea la acción del LLM

    # 4. Éxito: el flujo continúa
    print("Guardrail passed: Validación superada. Los parámetros son seguros.")
    sys.exit(0)

if __name__ == "__main__":
    main()
```