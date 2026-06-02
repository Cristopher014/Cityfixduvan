# Auditoría QA - CityFixDuván

## Introducción
Este documento resume la auditoría técnica realizada sobre el proyecto CityFixDuván, enfocada en la infraestructura Docker y la ejecución de pruebas E2E.

## Diagnóstico
- La imagen se construyó correctamente (`cityfixduvan-cityfix-app:latest`) y el contenedor `cityfix-container` se inició.
- Sin embargo, al intentar levantar servicios con `docker compose up -d app` o `docker compose up -d cityfix-container`, se reporta:  
  **"no such service"**.
- Al ejecutar pruebas con `docker compose exec app npm test` o `docker compose exec cityfix-app npm test`, se obtiene:  
  **"service 'app' is not running"**.
- Al usar `docker compose run --rm cityfix-app npm test`, el contenedor se crea pero falla con:  
  **"sh: jest: not found"**.

## Arquitectura
- Infraestructura basada en Docker Compose con un servicio esperado (`app` o `cityfix-app`).
- Se construye la imagen desde `node:20-alpine`.
- Se instalan dependencias con `npm install` sin vulnerabilidades reportadas.
- El contenedor se mantiene vivo con `tail -f /dev/null`.

## Problemas Detectados
- **Nombre de servicio inconsistente**: se intentó usar `app`, `cityfix-app`, `cityfix-container` y `cityfix`, pero ninguno coincide con el servicio definido en `docker-compose.yml`.
- **Confusión entre `container_name` y nombre de servicio**: el contenedor se llama `cityfix-container`, pero Compose espera el identificador del servicio.
- **Pruebas bloqueadas**: `jest` no está instalado dentro de la imagen, provocando el error `sh: jest: not found`.

## Recomendaciones
- Revisar el archivo `docker-compose.yml` y confirmar el nombre exacto del servicio bajo `services:`.
- Evitar usar `container_name` para que Compose gestione los nombres automáticamente.
- Instalar `jest` en la imagen o añadirlo como dependencia en `package.json`.
- Usar `docker compose ps` para verificar qué servicios están activos antes de ejecutar `exec`.

## Conclusión
El proyecto CityFixDuván logra construir y levantar el contenedor, pero presenta errores de configuración en los nombres de servicio y falta de dependencias para ejecutar pruebas. Corrigiendo la definición de servicios y asegurando la instalación de `jest`, el flujo debería funcionar correctamente.
